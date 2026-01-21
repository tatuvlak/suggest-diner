# Suggest Diner

An LLM-powered Android mobile application that helps users decide what to cook for dinner by learning their preferences, tracking their pantry inventory, and enabling family collaboration in meal planning.

## 📱 Overview

Suggest Diner takes the stress out of daily meal planning by combining artificial intelligence, smart inventory management, and family collaboration features to make dinner decisions easy and fun.

## 🎯 Core Features

- **🤖 AI-Powered Suggestions**: Get personalized dinner recommendations using LLM technology (GPT-4, Claude, Gemini)
- **👤 Preference Learning**: The app learns your tastes, dietary restrictions, and cooking preferences over time
- **🎲 "Go Crazy" Mode**: Explore new cuisines and recipes outside your comfort zone
- **🧊 Smart Pantry Management**: Track ingredients, expiry dates, and get suggestions based on what you have
- **👨‍👩‍👧‍👦 Family Polling**: Let the whole family vote on what's for dinner
- **📅 Meal Planning**: Plan your week ahead with AI assistance
- **🛒 Smart Shopping Lists**: Auto-generate shopping lists from selected recipes

## 📚 Documentation

### Design Documents
- **[DESIGN.md](./DESIGN.md)** - Comprehensive feature design, data models, UX/UI design, and product strategy
- **[TECHNICAL_ARCHITECTURE.md](./TECHNICAL_ARCHITECTURE.md)** - System architecture, API design, database schemas, and deployment strategy

### Quick Links to Key Sections

#### From DESIGN.md
- [Core Features](./DESIGN.md#1-core-features) - LLM suggestions, preference learning, "Go Crazy" mode, smart pantry, family polls
- [Additional Features](./DESIGN.md#2-additional-useful-features) - Meal planning, cooking assistance, social features, health tracking
- [User Interface Design](./DESIGN.md#5-user-interface--ux-design) - Screen designs and UX principles
- [Privacy & Security](./DESIGN.md#6-privacy--security-considerations) - Data protection and compliance
- [Monetization Strategy](./DESIGN.md#7-monetization-strategy-optional) - Free vs. Premium features
- [Development Phases](./DESIGN.md#11-development-phases) - MVP to full launch roadmap

#### From TECHNICAL_ARCHITECTURE.md
- [System Architecture](./TECHNICAL_ARCHITECTURE.md#1-system-architecture-diagram) - High-level component overview
- [Database Schema](./TECHNICAL_ARCHITECTURE.md#3-database-schema) - PostgreSQL, MongoDB, and Redis designs
- [API Endpoints](./TECHNICAL_ARCHITECTURE.md#4-api-endpoints) - Complete REST API specification
- [Security Implementation](./TECHNICAL_ARCHITECTURE.md#5-security-implementation) - Authentication, encryption, validation
- [Deployment Strategy](./TECHNICAL_ARCHITECTURE.md#6-deployment-architecture) - Cloud infrastructure and CI/CD

## 🏗️ Technology Stack

### Mobile App (Android)
- **Language**: Kotlin
- **UI Framework**: Jetpack Compose
- **Architecture**: MVVM (Model-View-ViewModel)
- **Local Database**: Room (SQLite)
- **Networking**: Retrofit + OkHttp
- **Dependency Injection**: Hilt/Dagger
- **Camera/Barcode**: CameraX + ML Kit

### Backend Services
- **API Framework**: Node.js/Python/Kotlin (Spring Boot)
- **Database**: PostgreSQL + MongoDB + Redis
- **LLM Integration**: OpenAI GPT-4, Anthropic Claude, Google Gemini
- **Cloud**: AWS/GCP/Azure
- **Containers**: Docker + Kubernetes

### AI/ML
- **LLM APIs**: OpenAI, Anthropic, Google
- **ML Frameworks**: TensorFlow/PyTorch for preference learning
- **Natural Language**: Advanced prompt engineering for recipe generation

## 🎨 Key Design Highlights

### User Experience Principles
1. **Simplicity First**: Reduce decision fatigue with clear, actionable suggestions
2. **Speed**: Get to a suggestion in under 3 seconds
3. **Delight**: Micro-animations and encouraging feedback
4. **Accessibility**: Voice commands, large text, screen reader support

### Unique Differentiators
- **Family Collaboration**: Unique voting system for household meal decisions
- **"Go Crazy" Mode**: Encourages culinary exploration with controlled randomness
- **Smart Pantry**: Proactive suggestions before ingredients expire
- **Holistic Approach**: Combines planning, inventory, and social features

## 📊 Data Privacy & Security

- **GDPR Compliant**: Full user control over personal data
- **Encryption**: TLS in transit, AES-256 at rest
- **Authentication**: OAuth 2.0 with JWT tokens
- **Child Safety**: Parental controls for family features
- **Transparent**: Clear privacy policy and data usage

## 🚀 Development Roadmap

### Phase 1: MVP (3-4 months)
- Basic LLM-powered recipe suggestions
- User preferences and profiles
- Manual pantry management
- Core UI screens

### Phase 2: Smart Features (2-3 months)
- Family polling system
- Preference learning
- "Go Crazy" mode
- Barcode scanning

### Phase 3: Advanced Features (2-3 months)
- Nutritional tracking
- Budget management
- Community features
- Voice assistant

### Phase 4: Polish & Scale (2-3 months)
- Performance optimization
- Premium features
- Marketing launch

## 📈 Success Metrics

- Daily Active Users (DAU) and retention rates
- Recipe suggestion acceptance rate
- Family poll participation
- User satisfaction (NPS)
- Premium conversion rate

## 🌍 Future Enhancements

- Smart fridge hardware integration
- AR cooking guidance
- Meal kit service partnerships
- IoT kitchen appliance control
- International expansion

## 📝 Current Status

**Status**: Design Phase Complete ✅  
**Next Steps**: Create wireframes/mockups, set up development environment, begin MVP development

## 👥 Contributing

This project is currently in the design phase. Once development begins, we welcome contributions!

## 📄 License

TBD

---

**Version**: 1.0  
**Last Updated**: 2026-01-21  
**Documentation Status**: Ready for Development
