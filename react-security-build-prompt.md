React (SPA) frontend + [BACKEND FRAMEWORK'ÜNÜ BELİRT] backend için authentication güvenlik 
mimarisi kurmak istiyorum. Kod yazmaya başlamadan ÖNCE bana aşağıdaki soruları sor ve 
cevaplarıma göre uygun mimariyi seç. Varsayım yapma, sor.

=== BAŞLAMADAN ÖNCE SORULACAK SORULAR ===

1. PROJE BÜYÜKLÜĞÜ:
   - Küçük ölçek (tek geliştirici, az kullanıcı, MVP/prototip)
   - Orta ölçek (birkaç geliştirici, aktif kullanıcı kitlesi, büyüyen ürün)
   - Büyük ölçek (kurumsal, çok sayıda kullanıcı, yüksek güvenlik gereksinimi)

2. CACHE / REDİS KULLANIMI:
   - Redis (veya benzeri bir cache/in-memory store) kullanıyor muyum ya da kurmak ister miyim?
   - Yoksa sadece mevcut veritabanımı mı kullanmalıyız?

3. MEVCUT ALTYAPI:
   - Hangi veritabanını kullanıyorum? (PostgreSQL/MongoDB/MySQL vb.)
   - Backend framework'üm ne? (Express/NestJS/Django vb.)
   - Zaten bir authentication yapım var mı, yoksa sıfırdan mı kuracağız?

4. İLERİ SEVİYE İHTİYAÇLAR (büyük ölçekliyse sor):
   - Social login / OAuth2 (Google, Microsoft vb.) gerekiyor mu?
   - Multi-device oturum yönetimi ("tüm cihazlardan çıkış yap") gerekiyor mu?
   - Audit log / anomali tespiti gerekiyor mu?

=== CEVAPLARA GÖRE UYGULANACAK MİMARİ ===

--- KÜÇÜK ÖLÇEK seçilirse ---
- Token: localStorage'da JWT (basit access token, orta ömürlü ~1 saat)
- Frontend: ProtectedRoute + axios interceptor (401 yakalayıp login'e yönlendirme)
- Redis: kullanılmaz, gerek yok
- Refresh token: opsiyonel, yoksa süre dolunca direkt tekrar login

--- ORTA ÖLÇEK seçilirse ---
- Token: httpOnly + Secure + SameSite cookie'de access token (kısa ömürlü) + refresh token (uzun ömürlü)
- Refresh token rotation uygula
- CSRF koruması: Double Submit Cookie pattern
- Silent refresh: axios interceptor'da 401 → otomatik /refresh → orijinal isteği tekrarla (request queue ile race condition önle)
- Redis CEVABINA göre:
  - Redis VARSA: refresh token'ları ve blacklist'i Redis'te TTL ile yönet
  - Redis YOKSA: refresh_tokens ve blacklisted_tokens tablolarını ana veritabanında tut, 
    expiry alanıyla ve periyodik cron job (node-cron vb.) ile temizle
- Rate limiting: /login ve /refresh endpoint'lerine uygula (express-rate-limit vb.)

--- BÜYÜK ÖLÇEK seçilirse ---
- Orta ölçekteki her şeye ek olarak:
- Token revocation/blacklist mekanizması (Redis şiddetle önerilir, ama cevaba göre karar ver)
- Rol bazlı yetkilendirme (RoleBasedRoute component'i)
- Multi-device oturum yönetimi (kullanıcı aktif oturumlarını görüp sonlandırabilsin)
- OAuth2/OIDC + PKCE desteği (cevaba göre)
- Audit logging (login/logout/refresh/başarısız denemeler) (cevaba göre)
- Güvenlik header'ları (helmet.js) + CSP
- Anomali tespiti: aynı token'ın farklı IP'lerden kullanımını flag'leme (cevaba göre)

=== TESLİMAT ===
Seçilen ölçeğe göre:
1. Backend: gerekli middleware, route, servis dosyalarını oluştur
2. Frontend: AuthContext.jsx, ProtectedRoute.jsx, PublicRoute.jsx, api.js (axios instance + 
   interceptor) dosyalarını oluştur; büyük ölçekse RoleBasedRoute.jsx da ekle
3. Redis kullanılacaksa bağlantı/config dosyasını, kullanılmayacaksa veritabanı tablo 
   şemasını (migration) oluştur
4. .env dosyasında olması gereken secret/config değişkenlerini listele
5. Postman/curl ile nasıl test edileceğine dair kısa örnekler ekle

Mevcut proje klasör yapımı paylaşacağım, ona göre entegre et.
