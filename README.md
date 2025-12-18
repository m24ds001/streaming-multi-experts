# Streaming Multi-Source Expert Integration

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Node 18+](https://img.shields.io/badge/node-18+-green.svg)](https://nodejs.org/)

**Production-ready implementation of adaptive credibility assessment for streaming multi-source data integration.**

📄 **Paper:** "Streaming Multi-Source Integration Through Adaptive Credibility Assessment"  
👥 **Authors:** Ramsha Mehreen, T. Senthil Murugan  
🏫 **Institution:** Kakatiya Institute of Technology & Science, India  
📰 **Journal:** Knowledge-Based Systems (Elsevier) — Under Review

---

## 🎯 Overview

This repository contains the complete implementation of our adaptive credibility framework that:

- ✅ **67% error reduction** vs best single source (validated on 152.4M events)
- ✅ **Proven convergence** in 52-68 days with O(log 1/ε) guarantee
- ✅ **11.3-24.2× ROI** across production deployments
- ✅ **Sub-200ms latency** at 1,840 events/second
- ✅ **Cross-domain validated** in web analytics, medical, and financial domains

## 🚀 Quick Start

### Prerequisites

```bash
# Required
- Docker & Docker Compose
- Python 3.9+
- Node.js 18+

# Optional (for development)
- PostgreSQL 14+
- Redis 6+
```

### Installation (5 minutes)

```bash
# 1. Clone repository
git clone https://github.com/rmehreen/streaming-multi-expert.git
cd streaming-multi-expert

# 2. Start with Docker
docker-compose up -d

# 3. Access dashboard
open http://localhost:3000
```

That's it! The system is now running with simulated data sources.

### Quick Test

```bash
# Run synthetic benchmark (180 test cases)
npm run test:synthetic

# Expected output:
# ✓ MAE: 4.1%
# ✓ Convergence: 58 days
# ✓ Confidence calibration: ECE 0.031
```

## 📚 Documentation

- **[Setup Guide](docs/SETUP.md)** — Detailed installation instructions
- **[API Reference](docs/API.md)** — REST API documentation
- **[Architecture](docs/ARCHITECTURE.md)** — System design details
- **[Algorithms](docs/ALGORITHMS.md)** — Mathematical background
- **[Deployment](docs/DEPLOYMENT.md)** — Production deployment guide

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                     │
│          React Dashboard + Real-time Updates             │
├─────────────────────────────────────────────────────────┤
│                   Application Layer                      │
│   Orchestrator │ Blackboard │ Credibility Engine        │
│   Circuit Breakers │ Rate Limiters │ A/B Testing        │
├─────────────────────────────────────────────────────────┤
│                   Integration Layer                      │
│   GA4 │ Mixpanel │ PostHog │ Clarity │ Custom           │
├─────────────────────────────────────────────────────────┤
│                   Persistence Layer                      │
│   PostgreSQL │ Redis Cache │ S3 Archive                 │
└─────────────────────────────────────────────────────────┘
```

## 🔬 Core Algorithms

### Algorithm 1: Adaptive Credibility Learning

**Proven O(log 1/ε) convergence** (Theorem 1 in paper)

```python
from src.algorithms import AdaptiveCredibilityTracker

# Initialize with sources
tracker = AdaptiveCredibilityTracker(['ga4', 'mixpanel', 'posthog'])

# Record measurements
tracker.record_measurement(
    source_id='ga4',
    latency_ms=180,
    error=False,
    value=12500
)

# Update credibility (runs automatically every hour)
credibility = tracker.update_credibility()
# {'ga4': 0.94, 'mixpanel': 0.86, 'posthog': 0.89}
```

**Update Rule:**
```
θ_i(t+1) = ⎧ 0.88 × θ_i(t)              if error/timeout
           ⎨ 0.94 × θ_i(t)              if inconsistent
           ⎩ min(1, θ_i(t) + 0.012)     if consistent
```

### Algorithm 2: Multi-Source Aggregation

**O(m·n) complexity with early termination**

```python
from src.algorithms import MultiSourceAggregator

# Initialize aggregator
aggregator = MultiSourceAggregator(
    sources=['ga4', 'mixpanel', 'posthog'],
    credibility_scores={'ga4': 0.94, 'mixpanel': 0.86, 'posthog': 0.89}
)

# Query metrics
result = await aggregator.aggregate({
    'metric': 'pageviews',
    'date_range': {'start': '2024-01-01', 'end': '2024-01-31'}
})

# Result:
# {
#   'value': 125430,
#   'confidence': 0.92,
#   'sources_used': ['ga4', 'mixpanel', 'posthog'],
#   'latency_ms': 247
# }
```

**Variance-Calibrated Confidence:**
```
c = (1 - σ(V) / (max(V) + ε)) × min(θ_i)
```

## 💻 Usage Examples

### Basic Integration

```javascript
const { MultiSourceIntegrator } = require('./src');

const integrator = new MultiSourceIntegrator({
  providers: {
    ga4: { apiKey: process.env.GA4_API_KEY },
    mixpanel: { token: process.env.MIXPANEL_TOKEN },
    posthog: { apiKey: process.env.POSTHOG_KEY }
  }
});

// Query with automatic aggregation
const metrics = await integrator.query({
  metrics: ['pageviews', 'sessions', 'bounce_rate'],
  dateRange: { start: '2024-01-01', end: '2024-01-31' }
});

console.log(metrics);
// {
//   pageviews: { value: 125430, confidence: 0.94 },
//   sessions: { value: 34520, confidence: 0.89 },
//   bounce_rate: { value: 0.32, confidence: 0.91 }
// }
```

### Real-Time Monitoring

```javascript
// Subscribe to credibility updates
integrator.on('credibility:update', (event) => {
  console.log(`${event.provider}: ${event.credibility.toFixed(3)}`);
  // ga4: 0.941
  // mixpanel: 0.863
});

// Subscribe to incidents
integrator.on('incident:detected', (event) => {
  console.log(`⚠️  ${event.type} on ${event.provider}`);
  console.log(`   Impact: ${event.impact.mae}% MAE increase`);
  // ⚠️  schema_migration on posthog
  //    Impact: 18.4% MAE increase
});
```

### Custom Provider Adapter

```javascript
class MyAnalyticsAdapter extends BaseAdapter {
  async query(params) {
    const response = await this.api.get('/analytics', params);
    return this.normalize(response);
  }
  
  normalize(data) {
    return {
      metrics: { pageviews: data.pv, sessions: data.sess },
      timestamp: Date.now(),
      source: 'my_analytics'
    };
  }
}

integrator.registerProvider('my_analytics', MyAnalyticsAdapter);
```

## 🐳 Docker Deployment

### Development

```bash
docker-compose up
```

### Production

```bash
docker-compose -f docker-compose.prod.yml up -d
```

### Environment Variables

```bash
# Required
DATABASE_URL=postgresql://user:pass@localhost/streaming_expert
REDIS_URL=redis://localhost:6379

# Provider API Keys
GA4_PROPERTY_ID=your_property_id
GA4_API_KEY=your_api_key
MIXPANEL_PROJECT_TOKEN=your_token
POSTHOG_API_KEY=your_key

# Optional
PORT=3000
LOG_LEVEL=info
CACHE_TTL=300
```

## 📊 Performance Benchmarks

| Metric | Value |
|--------|-------|
| **Latency (p50)** | 182ms |
| **Latency (p95)** | 247ms |
| **Latency (p99)** | 389ms |
| **Throughput** | 1,840 events/sec |
| **Memory** | 512MB avg |
| **CPU** | 2 cores |
| **Power** | 47W avg |
| **Cache Hit Rate** | 74% |

## 🧪 Testing

```bash
# Run all tests
npm test

# Unit tests only
npm run test:unit

# Integration tests (requires Docker)
npm run test:integration

# Synthetic benchmarks
npm run test:synthetic

# Coverage report
npm run test:coverage
```

## 📖 Reproducing Paper Results

### Main Evaluation (15 sites, 250 days)

```bash
python scripts/reproduce_main_evaluation.py \
  --config config/paper_reproduction.yml \
  --output results/main_evaluation/

# Generates all paper figures and tables
```

### Cross-Domain Validation

```bash
# Medical (MIMIC-III)
python scripts/reproduce_medical.py \
  --data data/mimic3_hr_data.csv \
  --output results/medical/

# Financial (market data)
python scripts/reproduce_financial.py \
  --data data/market_data.csv \
  --output results/financial/
```

## 📄 Citation

If you use this code in your research, please cite our paper:

```bibtex
@article{mehreen2025streaming,
  title={Streaming Multi-Source Integration Through Adaptive Credibility Assessment},
  author={Mehreen, Ramsha and Murugan, T. Senthil},
  journal={Knowledge-Based Systems},
  year={2025},
  publisher={Elsevier},
  note={Under Review}
}
```

## 🤝 Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📝 License

This project is licensed under the Apache License 2.0 - see [LICENSE](LICENSE) for details.

## 💬 Support

- **Issues:** [GitHub Issues](https://github.com/rmehreen/streaming-multi-expert/issues)
- **Email:** m24ds001@kitsw.ac.in
- **Paper:** [arXiv preprint](https://arxiv.org/abs/XXXX.XXXXX) (coming soon)

## 🗺️ Roadmap

### v1.1 (Q2 2025)
- [ ] Automated schema induction
- [ ] Transfer learning for faster convergence
- [ ] Additional provider adapters (Adobe Analytics, Amplitude)

### v2.0 (Q3 2025)
- [ ] Privacy-preserving architectures
- [ ] Federated credibility learning
- [ ] Formal verification tools

## 🙏 Acknowledgments

- Kakatiya Institute of Technology & Science for infrastructure
- Participating organizations for deployment access
- AWS Educate for computing credits
- Domain experts for evaluation feedback

---

**⭐ Star this repo if you find it useful!**

**Questions?** Open an issue or email m24ds001@kitsw.ac.in
