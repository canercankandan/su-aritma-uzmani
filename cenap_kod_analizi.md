# Cenap Su Arıtma - Kod Analizi

## 🎯 Proje Genel Bakış

**Cenap Su Arıtma**, Go programlama dili ve Gin framework'ü kullanılarak geliştirilmiş modern bir e-ticaret web uygulamasıdır. Su arıtma cihazları satışı için tasarlanmış, tam özellikli bir online mağaza sistemidir.

### 🚀 Ana Özellikler
- **E-ticaret**: Ürün katalogu, sepet sistemi, sipariş yönetimi
- **Kullanıcı Yönetimi**: Kayıt, giriş, profil yönetimi, şifre sıfırlama
- **Admin Paneli**: Ürün, sipariş ve kullanıcı yönetimi
- **Canlı Destek**: WebRTC tabanlı video görüşme özelliği
- **E-posta Bildirimleri**: SMTP entegrasyonu
- **Güvenlik**: HTTPS, SSL sertifikaları, şifre hashleme

## 🏗️ Teknik Mimari

### Backend Stack
- **Go 1.21+** - Ana programlama dili
- **Gin Framework** - HTTP web framework
- **JSON Database** - Dosya tabanlı veri saklama
- **bcrypt** - Şifre hashleme
- **SMTP/Gmail** - E-posta servisleri
- **WebRTC** - Video görüşme altyapısı

### Frontend Stack
- **HTML5/CSS3** - Markup ve stil
- **JavaScript** - İstemci tarafı mantık
- **Go Templates** - Server-side rendering
- **Bootstrap** - Responsive tasarım

## 📁 Proje Yapısı Analizi

### 1. Ana Dizin Yapısı
```
workspace/
├── cenap/cmd/web/         # Boş main.go (aktif değil)
├── cmd/web/               # Ana uygulama giriş noktası
├── internal/              # İç paketler
│   ├── database/          # Veritabanı katmanı
│   ├── handlers/          # HTTP handler'ları
│   ├── models/            # Veri modelleri
│   └── services/          # İş mantığı servisleri
├── templates/             # HTML şablonları
├── static/                # CSS, JS, resimler
├── data.json              # Ana veritabanı dosyası
├── orders.json            # Sipariş verileri
└── main.exe               # Derlenmiş uygulama
```

### 2. Veri Modelleri (internal/models/)

#### Product Model
```go
type Product struct {
    ID          int       `json:"id"`
    Name        string    `json:"name"`
    Description string    `json:"description"`
    Price       float64   `json:"price"`
    Image       string    `json:"image"`
    Category    string    `json:"category"`
    Stock       int       `json:"stock"`
    Features    string    `json:"features"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}
```

#### User Model
```go
type User struct {
    ID           int       `json:"id"`
    FullName     string    `json:"full_name"`
    Username     string    `json:"username"`
    Email        string    `json:"email"`
    PasswordHash string    `json:"password_hash"`
    ResetToken   string    `json:"reset_token,omitempty"`
    ResetExpiry  time.Time `json:"reset_expiry,omitempty"`
    CreatedAt    time.Time `json:"created_at"`
    UpdatedAt    time.Time `json:"updated_at"`
}
```

#### Order Model
```go
type Order struct {
    ID           int        `json:"id"`
    UserID       int        `json:"user_id"`
    SessionID    string     `json:"session_id"`
    OrderNumber  string     `json:"order_number"`
    CustomerName string     `json:"customer_name"`
    Email        string     `json:"email"`
    Phone        string     `json:"phone"`
    Address      string     `json:"address"`
    Items        []CartItem `json:"items"`
    TotalPrice   float64    `json:"total_price"`
    Status       string     `json:"status"`
    PaymentMethod string    `json:"payment_method"`
    Notes        string     `json:"notes"`
    AdminNotes   string     `json:"admin_notes"`
    CreatedAt    time.Time  `json:"created_at"`
    UpdatedAt    time.Time  `json:"updated_at"`
}
```

## 🔧 Veritabanı Katmanı Analizi

### JSON Tabanlı Veritabanı
Proje, geleneksel SQL veritabanı yerine **JSON dosya tabanlı** bir sistem kullanıyor:

#### Avantajları:
- ✅ Kolay kurulum (dış bağımlılık yok)
- ✅ Hızlı prototip geliştirme
- ✅ Basit backup ve restore

#### Dezavantajları:
- ❌ Büyük veri setlerinde performans sorunu
- ❌ Eşzamanlı yazma işlemlerinde risk
- ❌ Karmaşık sorgulama kabiliyeti sınırlı

### Veri Yapısı
```go
type dbData struct {
    Products          []models.Product          `json:"products"`
    Users             []models.User             `json:"users"`
    Orders            []models.Order            `json:"orders"`
    Messages          []models.Message          `json:"messages"`
    SupportSessions   []models.SupportSession   `json:"support_sessions"`
    VideoCallRequests []models.VideoCallRequest `json:"video_call_requests"`
}
```

### Thread Safety
```go
type JSONDatabase struct {
    mu       sync.RWMutex  // Okuma/yazma kilidi
    data     dbData
    filePath string
}
```

## 🚦 HTTP Routing Analizi

### Ana Sayfa Routes
```go
r.GET("/", h.HomePage)
r.GET("/products", h.ProductsPage)
r.GET("/about", h.AboutPage)
r.GET("/contact", h.ContactPage)
```

### E-ticaret Routes
```go
// Sepet İşlemleri
r.GET("/cart", h.CartPage)
r.POST("/cart/add", h.AddToCart)
r.POST("/cart/update", h.UpdateCartItem)
r.POST("/cart/remove", h.RemoveFromCart)

// Sipariş İşlemleri
r.GET("/checkout", h.CheckoutPage)
r.POST("/checkout", h.HandleCheckout)
r.GET("/order-success", h.OrderSuccessPage)
```

### Kullanıcı Kimlik Doğrulama
```go
// Public Routes
r.GET("/login", h.LoginPage)
r.POST("/login", h.HandleLogin)
r.GET("/register", h.RegisterPage)
r.POST("/register", h.HandleRegister)

// Protected Routes
user := r.Group("/profile")
user.Use(h.AuthUserMiddleware())
{
    user.GET("", h.ProfilePage)
    user.POST("/change-password", h.HandleChangePassword)
}
```

### Admin Paneli (Korumalı)
```go
admin := r.Group("/admin")
admin.Use(h.AuthMiddleware())
{
    admin.GET("", h.AdminPage)
    admin.POST("/add-product", h.AddProduct)
    admin.GET("/orders", h.AdminGetOrders)
    admin.GET("/support", h.AdminSupportPage)
}
```

## 🎮 Canlı Destek Sistemi

### WebRTC Video Görüşme
Proje, modern **WebRTC** teknolojisini kullanarak müşteri-admin arasında video görüşme özelliği sunuyor:

```go
// Public routes
r.GET("/support", h.SupportChatPage)
r.POST("/support/send", h.SendSupportMessage)
r.POST("/support/video-call-request", h.HandleVideoCallRequest)
r.POST("/support/webrtc-signal", h.HandleWebRTCSignal)

// Admin routes
admin.POST("/support/video-call-response", h.AdminVideoCallResponse)
admin.POST("/support/start-video-call", h.AdminStartVideoCall)
```

### Mesajlaşma Sistemi
```go
type Message struct {
    ID        int       `json:"id"`
    SessionID string    `json:"session_id"`
    Username  string    `json:"username"`
    Content   string    `json:"content"`
    IsAdmin   bool      `json:"is_admin"`
    CreatedAt time.Time `json:"created_at"`
}
```

## 📧 E-posta Servisleri

### SMTP Konfigürasyonu
```go
type EmailService struct {
    dialer *gomail.Dialer
    from   string
}

// Gmail SMTP
smtpHost := "smtp.gmail.com"
smtpPort := 587
smtpUser := os.Getenv("SMTP_USER")
smtpPass := os.Getenv("SMTP_PASS")
```

### E-posta Tipleri
1. **Hoş Geldin E-postası** - Yeni kullanıcı kaydı
2. **Şifre Sıfırlama** - Parola sıfırlama token'ı
3. **Sipariş Bildirimi** - Admin'e yeni sipariş bildirimi
4. **Video Call Bildirimi** - Admin'e video görüşme talebi
5. **Canlı Destek Bildirimi** - Yeni ziyaretçi bildirimi

## 🔐 Güvenlik Özellikleri

### Şifre Güvenliği
```go
// bcrypt kullanarak şifre hashleme
hashedPassword, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)

// Şifre doğrulama
func CheckPasswordHash(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}
```

### SSL/TLS Desteği
```go
// Self-signed sertifika oluşturma
func generateSelfSignedCert() (tls.Certificate, error) {
    // RSA anahtar çifti oluştur
    priv, err := rsa.GenerateKey(rand.Reader, 2048)
    // X509 sertifikası oluştur
    template := x509.Certificate{
        SerialNumber: big.NewInt(1),
        Subject: pkix.Name{
            Organization: []string{"Cenap Water Filters"},
            Country:     []string{"TR"},
        },
        NotBefore:   time.Now(),
        NotAfter:    time.Now().Add(365 * 24 * time.Hour),
        KeyUsage:    x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
        ExtKeyUsage: []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
        IPAddresses: []net.IP{net.IPv4(127, 0, 0, 1), net.ParseIP("192.168.1.133")},
        DNSNames:    []string{"localhost", "*.localhost"},
    }
}
```

### Middleware Koruması
```go
func (h *Handler) AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Session kontrolü
        // Admin yetkisi kontrolü
        // Unauthorized durumunda yönlendirme
    }
}
```

## 🌐 Deployment ve DevOps

### Multi-Server Desteği
Uygulama hem **HTTP** (8080) hem de **HTTPS** (8081) portlarında çalışıyor:

```go
// HTTP Server
httpPort := "8080"
go func() {
    if err := http.ListenAndServe(":"+httpPort, r); err != nil {
        log.Fatalf("HTTP Server başlatılamadı: %v", err)
    }
}()

// HTTPS Server
httpsPort := "8081"
httpsServer := &http.Server{
    Addr:      ":" + httpsPort,
    Handler:   r,
    TLSConfig: tlsConfig,
}
if err := httpsServer.ListenAndServeTLS("", ""); err != nil {
    log.Fatalf("HTTPS Server başlatılamadı: %v", err)
}
```

### Hetzner Cloud Entegrasyonu
Proje **Hetzner Cloud** deployment için özel scriptler içeriyor:
- `deploy-hetzner.sh` - Otomatik deployment
- `hetzner-quick-start.md` - Hızlı başlangıç kılavuzu
- Nginx reverse proxy konfigürasyonu

### Docker Desteği
```dockerfile
FROM golang:1.24-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o main ./cmd/web

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/main .
COPY static ./static
COPY templates ./templates
CMD ["./main"]
```

## 📊 Performans ve Optimizasyon

### Template Yönetimi
```go
templates := map[string]*template.Template{}
templateFiles := map[string][]string{
    "home.html":    {"templates/home.html", "templates/base.html"},
    "products.html": {"templates/products.html", "templates/base.html"},
    // Her sayfa için ayrı template seti
}
```

### Static File Serving
```go
r.Static("/static", "./static")
r.GET("/favicon.ico", func(c *gin.Context) {
    c.File("./static/favicon.ico")
})
```

### Production Mode
```go
gin.SetMode(gin.ReleaseMode)
```

## 🚨 Tespit Edilen Sorunlar ve Öneriler

### 1. Cenap Klasörü Sorunu
**Sorun**: `cenap/cmd/web/main.go` dosyası boş ve kullanılmıyor.
**Çözüm**: Ana `cmd/web/main.go` dosyası kullanılmalı veya cenap klasörü silinmeli.

### 2. Veritabanı Skalabilite
**Sorun**: JSON tabanlı veritabanı büyük veri setlerinde performans sorunu yaratabilir.
**Öneri**: PostgreSQL/MySQL'e geçiş planlanmalı.

### 3. Güvenlik İyileştirmeleri
**Öneriler**:
- Rate limiting eklenmeli
- CSRF koruması güçlendirilmeli
- JWT token sistemi değerlendirilmeli

### 4. Error Handling
**Sorun**: Bazı yerlerde error handling eksik.
**Öneri**: Merkezi error handling middleware eklenmeli.

### 5. Logging
**Öneri**: Structured logging (logrus/zap) kullanılmalı.

## 🎯 Sonuç

Cenap Su Arıtma projesi, modern Go web development pratiklerini kullanan, iyi yapılandırılmış bir e-ticaret uygulamasıdır. Özellikle:

### ✅ Güçlü Yanları:
- Temiz kod yapısı ve modüler tasarım
- Kapsamlı e-ticaret özellikleri
- Modern WebRTC entegrasyonu
- SSL/HTTPS güvenlik desteği
- Email notification sistemi
- Admin panel yönetimi
- Docker ve cloud deployment hazırlığı

### 🔄 İyileştirme Alanları:
- Veritabanı katmanı modernizasyonu
- Test coverage artırılması
- API documentation eklenmesi
- Monitoring ve logging iyileştirmeleri

Proje, küçük-orta ölçekli su arıtma işletmeleri için tam özellikli bir e-ticaret çözümü sunmaktadır.