# 🤖 ML Model Security Scanner - GDG Project

A comprehensive educational security analysis tool for machine learning models, designed to detect and explain various AI security vulnerabilities in real-time.

## 🎯 Overview

This project implements a **real-time ML security scanning system** that combines GitHub Actions CI/CD integration with a full-stack web application. It detects four major classes of ML security threats and provides educational explanations at multiple technical levels.

### Key Features

- 🎯 **Backdoor/Trojan Detection** - Identifies hidden triggers in model behavior
- 📋 **Model Extraction Risk Analysis** - Detects vulnerability to model cloning attacks
- 🔓 **Privacy Leakage Detection** - Checks for membership inference and overfitting risks
- ⚠️ **Unsafe Serialization Scanning** - Finds remote code execution vulnerabilities
- 📚 **Multi-level Explanations** - Beginner, Intermediate, Advanced learning materials
- 🔄 **Real-time Analysis** - Instant results with live API integration
- 📊 **Interactive Dashboard** - Beautiful React UI with risk visualization
- 📤 **Report Export** - Download security reports as JSON

---

## 🏗️ Architecture

```
gdg_project/
├── backend/                          # FastAPI application
│   ├── main.py                       # Entry point
│   ├── requirements.txt              # Python dependencies
│   ├── routes/
│   │   └── ml_security.py           # ML security endpoints
│   ├── ml_security/                 # ML security modules
│   │   ├── __init__.py
│   │   ├── backdoor_detector.py     # Backdoor analysis
│   │   ├── model_extraction.py      # Extraction risk analysis
│   │   ├── membership_inference.py  # Privacy leakage detection
│   │   ├── serialization_scanner.py # Code execution scanning
│   │   └── explainer.py             # Multi-level explanations
│   └── .env                          # Environment variables
│
├── frontend/                         # React + TypeScript
│   ├── src/
│   │   ├── components/
│   │   │   ├── MLSecurityDashboard.jsx   # Main dashboard
│   │   │   ├── MLSecurityDashboard.css   # Dashboard styles
│   │   │   └── ScanResults.tsx           # Results display
│   │   ├── hooks/
│   │   │   └── useMlSecurity.ts          # API integration hook
│   │   └── types/
│   │       └── scan.ts                    # TypeScript types
│   └── package.json
│
├── .github/
│   └── workflows/
│       └── security-scan.yml         # CI/CD pipeline
│
└── README.md                          # This file
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- Node.js 16+
- npm or yarn
- Git

### Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run FastAPI server
python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The API will be available at `http://localhost:8000`

### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

The dashboard will be available at `http://localhost:3000` (or shown in terminal)

---

## 🔧 API Endpoints

### Backdoor Detection

```bash
POST /api/v1/scan/backdoor
Content-Type: application/json

{
  "model_activations": {
    "layer1": [[0.1, 0.2], [0.3, 0.4]],
    "layer2": [[0.5, 0.6], [0.7, 0.8]]
  }
}
```

**Response:**
```json
{
  "risk_score": 0.45,
  "risk_level": "Medium",
  "indicators": ["Layer 'layer1': High activation anomaly detected"],
  "explanation": "Model shows some abnormal activation patterns...",
  "limitations": "🔬 EDUCATIONAL PoC LIMITATIONS: ..."
}
```

### Model Extraction Risk

```bash
POST /api/v1/scan/extraction
Content-Type: application/json

{
  "is_public": true,
  "has_auth": false,
  "returns_full_probs": true,
  "rate_limit": null
}
```

### Membership Inference Risk

```bash
POST /api/v1/scan/membership_inference
Content-Type: application/json

{
  "train_accuracy": 0.98,
  "test_accuracy": 0.75,
  "train_loss": 0.05,
  "test_loss": 0.3,
  "output_confidence": [0.98, 0.99, 0.97]
}
```

### Serialization Scanning

```bash
POST /api/v1/scan/serialization
Content-Type: application/json

{
  "repo_path": "."
}
```

### Get Explanations

```bash
GET /api/v1/explain/backdoor?level=beginner
GET /api/v1/explain/extraction?level=intermediate
GET /api/v1/explain/membership_inference?level=advanced
```

---

## 📚 Understanding ML Security Threats

### 🎯 Backdoor/Trojan Attacks

**What is it?**
A backdoor is a hidden trigger in a model that causes incorrect behavior. Like a stop sign that only fools a car from a specific angle.

**Example:**
- Attacker adds trigger pattern to training data paired with wrong label
- Model learns this backdoor along with normal behavior
- Works fine normally, but fails when trigger is present

**Why dangerous?**
- Difficult to detect (model appears normal in testing)
- Can be used for sabotage or espionage
- Undermines trust in AI systems

**Detection approach:**
- Analyze neuron activation patterns
- Look for abnormal statistical distributions
- Compare clean vs perturbed inputs

---

### 📋 Model Extraction Attacks

**What is it?**
Someone clones your model by querying it thousands of times and training a substitute.

**Real-world impact:**
- Loss of intellectual property
- Competitive disadvantage
- Training cost theft (you paid to train, they use for free)

**Attack steps:**
1. Attacker probes public endpoint with varied inputs
2. Collects thousands of predictions with confidence scores
3. Uses predictions to train substitute model
4. Cloned model mimics original behavior

**Defenses:**
- Require authentication (API keys)
- Rate limiting (100 requests/hour per user)
- Return only top-k predictions (not full softmax)
- Add noise to confidence scores
- Monitor for suspicious query patterns

---

### 🔓 Privacy Leakage (Membership Inference)

**What is it?**
Determining whether specific person's data was in training set - a privacy violation.

**Why it matters:**
- Violates GDPR, CCPA, and privacy laws
- Sensitive data (health, financial) at high risk
- Breaks user trust

**How it works:**
- Overfitted models give high confidence on training data
- But low confidence on test data
- Attacker queries model with different inputs
- Builds distribution to identify members

**Indicators:**
- Large accuracy gap (train >> test)
- Very high average confidence (>95%)
- Loss divergence (test_loss >> train_loss)

---

### ⚠️ Unsafe Serialization (Code Execution)

**What is it?**
Loading untrusted Python pickle files can execute arbitrary code.

**Real exploit:**
```python
import pickle

# Attacker-crafted pickle with embedded command
model = pickle.load(open('malicious_model.pkl'))  # ← Code executes here!
# Attacker gains full system access
```

**Vulnerable patterns:**
- `pickle.load()` - loads untrusted Python objects
- `torch.load()` without `weights_only=True`
- `joblib.load()` without restrictions

**Safe alternatives:**
- Use JSON/YAML for config
- `torch.load(..., weights_only=True)`
- ONNX format (no code execution)
- SafeTensors library
- Protocol Buffers

---

## 🎓 Educational Focus

This project is designed to teach AI security concepts:

1. **Beginner Level** - Simple analogies and real-world examples
2. **Intermediate Level** - Technical details and attack mechanisms
3. **Advanced Level** - Research papers, formal definitions, implementation details

Each vulnerability includes:
- Clear explanation at multiple levels
- Real attack scenarios
- Practical defenses
- Limitations and caveats

---

## 🔄 GitHub Actions Integration

The project includes a GitHub Actions workflow for automated scanning:

```yaml
# Triggered on:
# - Pull requests to main/develop
# - Pushes to main

# Scans for:
# - Traditional code vulnerabilities
# - ML-specific security issues
# - Posts results to PR as comment
```

**Usage:**
```bash
git push origin feature-branch
# Creates PR → GitHub Actions runs security scan → Results posted automatically
```

---

## 📊 Dashboard Features

### Real-time Scanning
- Select scan type from sidebar
- Click "Run Scan" to execute
- View results instantly as they arrive

### Attack Categories Filter
- Filter by threat type
- View specific vulnerability details
- Compare risk scores

### Risk Visualization
- Color-coded severity badges
- Risk score meters (0-100%)
- Summary statistics

### Explainability Drawer
- Click any vulnerability for details
- Switch between explanation levels
- View indicators and mitigations
- Educational disclaimers

### Report Export
- Download as JSON
- Share with team
- Archive for compliance

---

## 🛠️ Development

### Project Structure

```
Frontend: React 18 + TypeScript + Tailwind CSS
Backend: FastAPI + NumPy
CI/CD: GitHub Actions
```

### Key Technologies

**Backend:**
- FastAPI - Modern web framework
- NumPy - Numerical analysis
- Pydantic - Data validation

**Frontend:**
- React - UI library
- TypeScript - Type safety
- Lucide Icons - Icons
- CSS - Custom styling

### Running Tests

```bash
# Backend tests
cd backend
pytest

# Frontend tests
cd frontend
npm run test
```

---

## ⚖️ Limitations & Disclaimers

### Educational Tool
This is a **proof-of-concept for educational purposes**. Not a production-grade security tool.

### Heuristic-based Detection
- Uses statistical analysis, not formal verification
- False positives and false negatives possible
- Backdoors can be designed to evade detection
- Not a guarantee of safety

### Requires Access
- Needs internal model activations (sometimes unavailable)
- Requires code to scan for unsafe patterns
- API endpoint configuration must be provided

### Best Practices
- Use alongside other defenses
- Validate training data
- Use clean datasets
- Regular security audits
- Monitor model behavior in production

---

## 🔒 Security Best Practices

### For Your Models

1. **Training Data Validation**
   - Audit data sources
   - Check for poisoning
   - Use data provenance tracking

2. **Model Hygiene**
   - Train on clean data
   - Use regularization to prevent overfitting
   - Apply early stopping

3. **Deployment Security**
   - Require authentication for APIs
   - Implement rate limiting
   - Return limited predictions (not full softmax)
   - Monitor for extraction attempts
   - Use differential privacy

4. **Serialization Safety**
   - Never load untrusted models
   - Use `weights_only=True` for PyTorch
   - Switch to ONNX or SafeTensors
   - Validate file signatures

---

## 📖 Learning Resources

### About Backdoors
- Paper: "BadNets: Identifying Vulnerabilities in the Neuron Weights of Deep Networks"
- Paper: "Trojan Attacks on Neural Networks"
- Course: Fast.ai - Security for ML

### About Model Extraction
- Paper: "Stealing Machine Learning Models via Prediction APIs"
- Paper: "Model Extraction and Distillation"

### About Privacy
- Paper: "Membership Inference Attacks Against Machine Learning Models"
- Course: Andrew Ng - Privacy-Preserving ML

### About Safe Serialization
- OWASP: "Deserialization Cheat Sheet"
- Python: Official pickle documentation warnings

---

## 🤝 Contributing

We welcome contributions! Areas for improvement:

- [ ] Add more backdoor detection methods
- [ ] Implement formal verification techniques
- [ ] Add poisoning attack detection
- [ ] Create more educational materials
- [ ] Add multi-language support
- [ ] Implement model watermarking detection
- [ ] Add adversarial robustness testing

### How to Contribute

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

---

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

---

## 👥 Authors

**GDG Project Team**
- AI Security Education Initiative
- Built for learning and awareness

---

## 📞 Support

### Issues & Questions
- Open an issue on GitHub
- Check existing documentation
- Review educational materials

### Contact
- Email: support@gdgproject.dev
- Twitter: @GDGSecure
- Website: www.gdgproject.dev

---

## 🎓 Citation

If you use this project in research or education:

```bibtex
@software{gdg_ml_security_2024,
  title={ML Model Security Scanner - Educational Security Analysis Tool},
  author={GDG Project Team},
  year={2024},
  url={https://github.com/gdgproject/ml-security-scanner}
}
```

---

## 🚀 Roadmap

### Version 1.0 (Current)
- ✅ Backdoor detection
- ✅ Model extraction analysis
- ✅ Privacy leakage detection
- ✅ Serialization scanning
- ✅ Multi-level explanations
- ✅ Real-time dashboard

### Version 1.1 (Planned)
- [ ] Poisoning attack detection
- [ ] Adversarial robustness testing
- [ ] Model watermarking
- [ ] Supply chain security
- [ ] API security audit tools

### Version 2.0 (Future)
- [ ] Formal verification
- [ ] Certified defenses
- [ ] Multi-model analysis
- [ ] Continuous monitoring
- [ ] Team collaboration features

---

## 🌟 Acknowledgments

Built with ❤️ for the AI security education community.

Special thanks to:
- FastAPI community
- React community
- Security researchers
- Open source contributon
# gdg-project
