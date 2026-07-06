React (SPA) frontend + Node.js/Express (veya kullandığım backend) için kurumsal seviyede 
authentication güvenlik mimarisi kurmak istiyorum. Orta ve büyük ölçekli sistemlere uygun, 
localStorage yerine güvenli cookie tabanlı bir yapı istiyorum. Aşağıdaki tüm katmanları uygula:

=== 1. TOKEN STRATEJİSİ (Access + Refresh Token) ===
- Access token: kısa ömürlü (10-15 dk), JWT formatında
- Refresh token: uzun ömürlü (7 gün), rastgele üretilmiş opak bir string (JWT olmasına gerek yok)
- Access token'ı response body'de DEĞİL, httpOnly + Secure + SameSite=Strict cookie olarak set et
- Refresh token'ı ayrı bir httpOnly cookie'de, farklı bir path'te (örn. /api/auth/refresh) tut
- Frontend hiçbir zaman token'ı localStorage/sessionStorage'da tutmasın, JS ile token'a erişim olmasın

=== 2. REFRESH TOKEN ROTATION ===
- Her /refresh isteğinde: eski refresh token invalidate edilsin, yenisi üretilip cookie'ye set edilsin
- Refresh token'lar veritabanında (veya Redis'te) hash'lenerek saklansın, kullanıldıkça "used" işaretlensin
- Aynı refresh token ikinci kez kullanılmaya çalışılırsa (token theft belirtisi): o kullanıcının TÜM aktif oturumları/refresh token'ları iptal edilsin ve güvenlik logu tutulsun

=== 3. CSRF KORUMASI ===
- httpOnly cookie kullanıldığı için CSRF riskine karşı Double Submit Cookie pattern uygula:
  - Backend, ayrı bir CSRF token üretip JS'in okuyabileceği (httpOnly olmayan) bir cookie'de versin
  - Frontend bu CSRF token'ı her state-changing istekte (POST/PUT/DELETE) custom header olarak göndersin (örn. X-CSRF-Token)
  - Backend header'daki değer ile cookie'deki değeri karşılaştırıp doğrulasın
- SameSite=Strict (veya cross-domain gerekiyorsa Lax + ek CSRF katmanı) kullan

=== 4. SILENT REFRESH MEKANİZMASI (Frontend) ===
- Axios instance'ı withCredentials: true ile kur (cookie otomatik gitsin)
- Response interceptor: 401 alındığında otomatik olarak /refresh endpoint'ine istek atsın, başarılıysa orijinal isteği tekrar dene (request queue mantığıyla, aynı anda birden fazla 401 gelirse tek refresh isteği atılsın - race condition önlensin)
- Refresh de başarısızsa: kullanıcıyı logout yap, /login'e yönlendir

=== 5. AUTH STATE YÖNETİMİ (Frontend) ===
- AuthContext (veya Zustand) üzerinden isAuthenticated, user bilgisi yönetilsin
- Token frontend'de tutulmadığı için, uygulama açılışında /api/auth/me gibi bir endpoint'e istek atarak (cookie otomatik gönderilir) kullanıcının login durumu ve bilgisi çekilsin
- Bu kontrol tamamlanana kadar loading/splash ekranı gösterilsin

=== 6. ROUTE GUARD ===
- ProtectedRoute: isAuthenticated false ise /login'e yönlendirsin
- PublicRoute (GuestRoute): zaten login olmuş kullanıcı /login veya /register'a giderse /dashboard'a yönlendirsin
- Route bazlı yetkilendirme de istiyorum: kullanıcı rolüne göre (admin/user) erişim kısıtlaması yapan bir RoleBasedRoute component'i de ekle

=== 7. TOKEN REVOCATION / BLACKLIST ===
- Redis kullanarak logout olan veya şüpheli görülen access token'ların jti (JWT ID) değerini blacklist'e ekle
- Her istekte middleware, token'ın blacklist'te olup olmadığını kontrol etsin (kısa TTL ile, access token'ın ömrü kadar tutulması yeterli)

=== 8. OAuth2 / OIDC DESTEĞİ (opsiyonel ama ekle) ===
- Google/Microsoft ile giriş (Social Login) için Authorization Code Flow + PKCE akışını destekleyecek bir yapı kur
- Passport.js (Node için) veya benzeri bir kütüphane ile OIDC provider entegrasyonu için altyapı hazırla

=== 9. GÜVENLİK HEADER'LARI VE CSP ===
- Backend'de helmet.js (Express için) kullanarak güvenlik header'larını ayarla:
  - Content-Security-Policy
  - X-Frame-Options
  - X-Content-Type-Options
  - Strict-Transport-Security
- Frontend'de dangerouslySetInnerHTML kullanımı varsa DOMPurify ile sanitize et

=== 10. RATE LIMITING VE BRUTE-FORCE KORUMASI ===
- /login ve /refresh endpoint'lerine express-rate-limit (veya benzeri) ile rate limiting uygula
- Belirli sayıda başarısız login denemesinden sonra IP veya kullanıcı bazlı geçici kilitleme (account lockout) ekle

=== 11. AKTİF OTURUM YÖNETİMİ (Multi-Device) ===
- Kullanıcının aktif oturumlarını (cihaz/IP/tarayıcı bilgisiyle) veritabanında listeleyebileceği bir yapı kur
- Kullanıcı "tüm cihazlardan çıkış yap" veya belirli bir oturumu sonlandırabilsin

=== 12. LOGGING VE ANOMALİ TESPİTİ ===
- Login, logout, refresh, başarısız giriş denemeleri gibi olayları audit log olarak kaydet
- Aynı token/oturumun farklı IP'lerden aynı anda kullanılması gibi durumları tespit edip flag'leyecek basit bir kontrol mekanizması ekle

=== TESLİMAT ===
Lütfen:
1. Backend tarafında gerekli middleware, route ve servisleri (authMiddleware.js, tokenService.js, 
   refreshTokenRepository.js vb.) oluştur
2. Frontend tarafında AuthContext.jsx, ProtectedRoute.jsx, PublicRoute.jsx, RoleBasedRoute.jsx, 
   api.js (axios instance + interceptor) dosyalarını oluştur
3. Kullandığım veritabanını (PostgreSQL/MongoDB/MySQL - belirt) ve Redis'i bu yapıya uygun şekilde entegre et
4. .env dosyasında olması gereken tüm secret/config değişkenlerini listele
5. Bu yapının nasıl test edileceğine dair kısa bir açıklama ekle (Postman/curl örnekleri)

Mevcut proje yapımı (klasörler, kullandığım kütüphaneler, veritabanı) paylaşacağım, buna göre entegre et.
