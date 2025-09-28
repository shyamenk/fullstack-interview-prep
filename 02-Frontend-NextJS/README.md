# Next.js Interview Preparation

## 🎯 Core Next.js Concepts

### Framework Fundamentals
- [ ] **File-based Routing**
  - Pages directory structure
  - Dynamic routes with brackets
  - Nested routes and layouts
  - API routes in pages/api
- [ ] **App Router (Next.js 13+)**
  - App directory structure
  - Layouts and templates
  - Loading and error boundaries
  - Route groups and parallel routes

### Rendering Methods
- [ ] **Server-Side Rendering (SSR)**
  - getServerSideProps
  - Dynamic data fetching
  - SEO benefits and use cases
- [ ] **Static Site Generation (SSG)**
  - getStaticProps and getStaticPaths
  - Incremental Static Regeneration (ISR)
  - Performance benefits
- [ ] **Client-Side Rendering (CSR)**
  - useEffect for data fetching
  - SWR and React Query integration
  - Hybrid approaches

### Performance Optimization
- [ ] **Image Optimization**
  - next/image component
  - Automatic WebP conversion
  - Lazy loading and responsive images
- [ ] **Code Splitting**
  - Automatic code splitting
  - Dynamic imports
  - Bundle analyzer

## 🚀 Advanced Features

### Data Fetching Patterns
- [ ] **App Router Data Fetching**
  - Server components
  - Async components
  - Streaming with Suspense
- [ ] **Static Generation**
  - Build-time data fetching
  - Revalidation strategies
  - On-demand revalidation

### API Routes
- [ ] **REST API Development**
  - Request handling
  - Middleware patterns
  - Error handling
- [ ] **Route Handlers (App Router)**
  - GET, POST, PUT, DELETE
  - Request/Response objects
  - Middleware integration

### Middleware & Authentication
- [ ] **Middleware Functions**
  - Request interception
  - URL rewriting
  - Authentication checks
- [ ] **Authentication Patterns**
  - NextAuth.js integration
  - JWT handling
  - Protected routes

## 💻 Common Interview Questions

### Conceptual Questions
1. **What's the difference between SSR, SSG, and CSR in Next.js?**
2. **How does Next.js handle automatic code splitting?**
3. **Explain the difference between getServerSideProps and getStaticProps**
4. **What is Incremental Static Regeneration (ISR)?**
5. **How do you optimize images in Next.js?**

### Technical Implementation
1. **Build a dynamic blog with SSG**
2. **Implement authentication with middleware**
3. **Create API routes with proper error handling**
4. **Set up internationalization (i18n)**
5. **Optimize a Next.js app for performance**

### Performance Questions
1. **How would you improve Core Web Vitals in Next.js?**
2. **What's the difference between next/image and regular img tags?**
3. **How do you implement lazy loading in Next.js?**
4. **Explain Next.js bundle optimization strategies**

## 🛠️ Hands-on Projects

### Beginner Projects
- [ ] **Portfolio Website**
  - Static pages with SSG
  - Contact form with API routes
  - Image optimization
- [ ] **Blog with Markdown**
  - Dynamic routing
  - Static generation
  - SEO optimization

### Intermediate Projects
- [ ] **E-commerce Store**
  - Product catalog with SSG
  - Shopping cart with CSR
  - Checkout with API routes
- [ ] **Dashboard Application**
  - Authentication middleware
  - Protected routes
  - Real-time data updates

### Advanced Projects
- [ ] **Multi-tenant SaaS**
  - Dynamic subdomains
  - Per-tenant routing
  - Custom middleware
- [ ] **Social Media Platform**
  - Real-time features
  - Image uploads
  - Infinite scroll with ISR

## 📁 Project Structure Examples

### Pages Router Structure
```
pages/
├── _app.js
├── _document.js
├── index.js
├── about.js
├── blog/
│   ├── index.js
│   └── [slug].js
└── api/
    ├── auth/
    │   └── [...nextauth].js
    └── posts/
        └── [id].js
```

### App Router Structure
```
app/
├── layout.js
├── page.js
├── loading.js
├── error.js
├── blog/
│   ├── layout.js
│   ├── page.js
│   └── [slug]/
│       └── page.js
└── api/
    └── posts/
        └── route.js
```

## 🔧 Configuration & Optimization

### Next.js Config
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  images: {
    domains: ['example.com'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },
  compress: true,
  poweredByHeader: false,
  generateEtags: false,
  httpAgentOptions: {
    keepAlive: true,
  },
}

module.exports = nextConfig
```

### Performance Optimizations
- [ ] **Bundle Analysis**
  - @next/bundle-analyzer
  - Webpack Bundle Analyzer
  - Tree shaking optimization
- [ ] **Caching Strategies**
  - HTTP cache headers
  - CDN integration
  - Service worker caching

## 🎨 Styling Solutions

### CSS Options
- [ ] **CSS Modules**
  - Scoped styling
  - Component-level CSS
  - Dynamic classes
- [ ] **Styled Components**
  - CSS-in-JS
  - Server-side rendering
  - Theme integration
- [ ] **Tailwind CSS**
  - Utility-first approach
  - JIT compilation
  - Custom design system

### Styling Best Practices
```javascript
// CSS Modules
import styles from './Button.module.css'

export default function Button({ children, variant = 'primary' }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  )
}

// Styled Components
import styled from 'styled-components'

const StyledButton = styled.button`
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  background-color: ${props => props.primary ? '#007bff' : '#6c757d'};
`
```

## 🌐 Deployment & Production

### Deployment Platforms
- [ ] **Vercel (Recommended)**
  - Zero-configuration deployment
  - Automatic previews
  - Edge functions
- [ ] **Netlify**
  - Git-based workflow
  - Form handling
  - Split testing
- [ ] **AWS/Digital Ocean**
  - Custom server deployment
  - Docker containers
  - Load balancing

### Production Optimization
- [ ] **Environment Variables**
  - Runtime vs build-time variables
  - Security considerations
  - Multiple environments
- [ ] **Monitoring**
  - Web Vitals tracking
  - Error monitoring
  - Performance analytics

## 📊 Testing Strategies

### Testing Tools
- [ ] **Jest & React Testing Library**
  - Component testing
  - API route testing
  - Mock implementations
- [ ] **Cypress**
  - End-to-end testing
  - Visual regression testing
  - Performance testing
- [ ] **Playwright**
  - Cross-browser testing
  - API testing
  - Mobile testing

### Testing Examples
```javascript
// Component testing
import { render, screen } from '@testing-library/react'
import Home from '../pages/index'

describe('Home', () => {
  it('renders a heading', () => {
    render(<Home />)
    const heading = screen.getByRole('heading', {
      name: /welcome to next.js!/i,
    })
    expect(heading).toBeInTheDocument()
  })
})

// API route testing
import handler from '../pages/api/hello'
import { createMocks } from 'node-mocks-http'

describe('/api/hello', () => {
  it('returns a message', async () => {
    const { req, res } = createMocks({
      method: 'GET',
    })

    await handler(req, res)

    expect(res._getStatusCode()).toBe(200)
    expect(JSON.parse(res._getData())).toEqual({ name: 'John Doe' })
  })
})
```

## 🔐 Security Best Practices

### Security Considerations
- [ ] **Content Security Policy**
  - XSS protection
  - Script source restrictions
  - Image source policies
- [ ] **Authentication Security**
  - Secure cookie settings
  - CSRF protection
  - Rate limiting
- [ ] **API Security**
  - Input validation
  - Authorization checks
  - Error handling

## 📚 Learning Resources

### Official Documentation
- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Learn](https://nextjs.org/learn)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)

### Recommended Courses
- "Complete Next.js Developer" by Zero to Mastery
- "Next.js & React" by Maximilian Schwarzmüller
- "Next.js for Production" by Lee Robinson (Vercel)

### Community Resources
- [Next.js GitHub Discussions](https://github.com/vercel/next.js/discussions)
- [Next.js Subreddit](https://www.reddit.com/r/nextjs/)
- [Vercel Blog](https://vercel.com/blog)

## ✅ Progress Tracker

### Core Concepts
- [ ] File-based routing mastered
- [ ] SSR vs SSG vs CSR understood
- [ ] Data fetching patterns practiced
- [ ] API routes implemented

### Advanced Features
- [ ] App Router (Next.js 13+) explored
- [ ] Image optimization configured
- [ ] Performance monitoring set up
- [ ] SEO optimization applied

### Production Skills
- [ ] Deployment pipeline configured
- [ ] Environment management set up
- [ ] Testing strategies implemented
- [ ] Security measures applied

### Practical Experience
- [ ] 3+ Next.js projects completed
- [ ] Full-stack application built
- [ ] Performance optimization applied
- [ ] Production deployment achieved

**Target**: Master Next.js for production-ready applications and confident interview discussions about modern React frameworks