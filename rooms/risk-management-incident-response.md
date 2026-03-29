## Overview

This [room](https://tryhackme.com/room/seriskmanagement) introduces the fundamentals of risk management within an information security context, covering how organizations identify, assess, and respond to risks threatening the confidentiality, integrity, and availability of their assets.

---

## Key Concepts Covered

### Risk Fundamentals
- **Risk** is defined as the potential for loss or harm to an organization's information assets
- **Vulnerability** — a weakness an attacker could exploit to gain unauthorized access
- **Threat** — any potential cause of an unwanted incident
- **Asset** — anything of value to the organization that requires protection

### Risk Response Strategies
| Strategy | Description |
|----------|-------------|
| **Risk Avoidance** | Eliminating the activity that introduces the risk |
| **Risk Acceptance** | Acknowledging the risk and choosing to take no action |
| **Risk Reduction** | Implementing controls to lower the likelihood or impact |
| **Risk Transference** | Shifting risk to a third party (e.g., insurance, outsourcing) |

### Risk Assessment & Quantification
- **ALE (Annualized Loss Expectancy)** = SLE × ARO
  - SLE (Single Loss Expectancy) — expected loss from a single incident
  - ARO (Annualized Rate of Occurrence) — estimated frequency per year
- **Safeguard Value** = ALE (before) − ALE (after) − Annual Cost of Safeguard
  - A positive value indicates the control is worth implementing

### Frameworks Referenced
- **NIST SP 800-53** — Security and Privacy Controls for Information Systems
- **NIST CSF** — Identify, Protect, Detect, Respond, Recover
- **NIST SP 800-61** — Incident Response lifecycle: Preparation → Detection & Analysis → Containment → Recovery → Post-Incident Activity

### Risk Monitoring
- Monitoring involves tracking whether implemented controls are effectively reducing risk over time
- Includes keeping watch on internal control effectiveness and external factors such as new regulations and emerging threats

---

## Takeaways

- Risk management is not a one-time exercise — it's a continuous cycle of identification, assessment, response, and monitoring
- Quantifying risk with ALE gives organizations a financial basis for justifying security investments
- Understanding the difference between risk response strategies is foundational for any GRC or security analyst role
