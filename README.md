# 📚 ShareUpTime - Proje Dokümantasyon Merkezi

Bu repository, ShareUpTime sosyal medya platformunun tüm dokümantasyonunu içerir. Geliştiriciler için kapsamlı rehberler, API dokümantasyonu, mimari açıklamaları ve kurulum talimatları burada yer alır.

## 📋 Dokümantasyon İçeriği

### 🎯 Ana Dokümantasyon
| Dosya | Açıklama | İçerik |
|-------|----------|--------|
| **[DOCUMENTATION.md](./DOCUMENTATION.md)** | Kapsamlı proje dokümantasyonu | Teknik mimari, kurulum, API endpoints, sorun giderme |
| **[ARCHITECTURE.md](./ARCHITECTURE.md)** | Sistem mimarisi detayları | Mikroservisler, veritabanları, veri akışı, ölçeklenebilirlik |
| **[API_DOCUMENTATION.md](./API_DOCUMENTATION.md)** | API referans dokümantasyonu | Tüm endpoint'ler, request/response örnekleri, error handling |

### 🚀 Kurulum ve Deployment
| Dosya | Açıklama | İçerik |
|-------|----------|--------|
| **[KURULUM_REHBERI.md](./KURULUM_REHBERI.md)** | Detaylı kurulum rehberi | Sistem gereksinimleri, Docker kurulumu, manuel kurulum |
| **[DEPLOYMENT.md](./DEPLOYMENT.md)** | Production deployment rehberi | CI/CD, Docker, Kubernetes, monitoring |

### 🔒 Güvenlik ve Katkı
| Dosya | Açıklama | İçerik |
|-------|----------|--------|
| **[SECURITY.md](./SECURITY.md)** | Güvenlik politikası | Güvenlik açığı raporlama, responsible disclosure |
| **[CONTRIBUTING.md](./CONTRIBUTING.md)** | Katkıda bulunma rehberi | Code standards, PR süreci, geliştirme workflow |
| **[CHANGELOG.md](./CHANGELOG.md)** | Değişiklik geçmişi | Versiyon notları, yeni özellikler, bug fix'ler |

## 🏗️ ShareUpTime Proje Yapısı

ShareUpTime modern mikroservis mimarisi ile geliştirilmiş sosyal medya platformudur:

### Backend Mikroservisleri
- **API Gateway** (Port 3000) - Request routing, authentication
- **Auth Service** (Port 3001) - Kimlik doğrulama, JWT token yönetimi
- **User Service** (Port 3002) - Kullanıcı profilleri, sosyal graf (Neo4j)
- **Post Service** (Port 3003) - İçerik yönetimi (MongoDB)
- **Feed Service** (Port 3004) - Timeline oluşturma (Redis)
- **Media Service** (Port 3005) - Dosya yükleme, işleme (MinIO)
- **Notification Service** (Port 3006) - Gerçek zamanlı bildirimler (Kafka)

### Frontend Uygulamaları
- **Web App**: Next.js 15 + TypeScript + Tailwind CSS
- **Mobile App**: React Native + TypeScript (Android/iOS)

### Veritabanları & Altyapı
- **PostgreSQL** - Yapılandırılmış veri
- **MongoDB Atlas** - Esnek içerik depolama
- **Neo4j** - Sosyal graf, öneriler
- **Redis** - Önbellekleme, oturum depolama
- **Elasticsearch** - Arama indeksleme
- **MinIO** - Nesne depolama
- **Kafka** - Event streaming
- **Prometheus + Grafana** - Monitoring

## ✨ Özellikler
- 🔐 JWT tabanlı güvenli kimlik doğrulama
- 📱 Responsive web arayüzü (Next.js 15)
- 📲 Cross-platform mobil uygulama (React Native)
- 🏗️ Mikroservis mimarisi
- 🔄 Real-time bildirimler
- 📊 Monitoring ve logging
- 🐳 Docker containerization
- 🌐 Modern Türkçe arayüz

## 🚀 Hızlı Başlangıç

### 1️⃣ Dokümantasyonu İncele
```bash
# Ana proje dokümantasyonu
📖 DOCUMENTATION.md - Genel bakış ve kurulum

# Sistem mimarisi
🏛️ ARCHITECTURE.md - Teknik detaylar ve veri akışı

# API referansı
🔌 API_DOCUMENTATION.md - Endpoint'ler ve örnekler
```

### 2️⃣ Kurulum için
```bash
# Detaylı kurulum talimatları
📋 KURULUM_REHBERI.md - Adım adım kurulum

# Production deployment
🚀 DEPLOYMENT.md - Canlı ortam kurulumu
```

### 3️⃣ Geliştirme için
```bash
# Katkıda bulunma rehberi
🤝 CONTRIBUTING.md - Code standards ve PR süreci

# Güvenlik politikası
🔒 SECURITY.md - Güvenlik açığı raporlama
```

## 📞 Destek ve İletişim

### Ana Proje Repository
- **GitHub**: [shareuptime-social-platform](https://github.com/ruhaverse/shareuptime-social-platform)
- **Issues**: [Sorun bildirimi](https://github.com/ruhaverse/shareuptime-social-platform/issues)
- **Discussions**: [Topluluk forumu](https://github.com/ruhaverse/shareuptime-social-platform/discussions)

### Dokümantasyon Güncellemeleri
- Bu repository'deki dokümantasyon ana projeden senkronize edilir
- Dokümantasyon güncellemeleri için ana proje repository'sine PR gönderin
- Dokümantasyon hataları için bu repository'de issue açabilirsiniz

## 📈 Dokümantasyon Durumu

| Dokümantasyon | Durum | Son Güncelleme |
|---------------|-------|----------------|
| README.md | ✅ Güncel | 2024-12-10 |
| DOCUMENTATION.md | ✅ Güncel | 2024-12-10 |
| ARCHITECTURE.md | ✅ Güncel | 2024-12-10 |
| API_DOCUMENTATION.md | ✅ Güncel | 2024-12-10 |
| KURULUM_REHBERI.md | ✅ Güncel | 2024-12-10 |
| SECURITY.md | ✅ Güncel | 2024-12-10 |
| CONTRIBUTING.md | ✅ Güncel | 2024-12-10 |
| DEPLOYMENT.md | ✅ Güncel | 2024-12-10 |
| CHANGELOG.md | ✅ Güncel | 2024-12-10 |

---

**📚 ShareUpTime Dokümantasyon Merkezi** - Geliştiriciler için kapsamlı rehber ve referans kaynağı.

**Made with ❤️ by Ruhaverse Team**
