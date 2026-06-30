# Secure Development of AI Applications

**Course:** MSSE 642 – Software Assurance  
**Research Topic:** Secure Development of AI Applications  
**Group Members:** Abdullah Bahir, Emad Fattah, Shawn Wilkinson  
**Date:** June 27, 2026

---

## Introduction

Artificial Intelligence has become a core part of how modern software systems operate. AI-powered applications are now showing up everywhere — healthcare, finance, transportation, cybersecurity, education, and software engineering. More and more organizations are leaning on these technologies to automate repetitive tasks, support decision-making, and deliver personalized experiences to users. But that rapid adoption hasn't come without a cost. It has introduced a new set of security, privacy, and reliability challenges that traditional software development practices just weren't designed to handle.

Securing AI applications means going beyond the usual software assurance checklist. Developers have to think carefully about risks that are specific to AI systems — things like prompt injection, model manipulation, data poisoning, unauthorized access to training data, and the potential for sensitive information to leak through model outputs. These aren't hypothetical concerns; they are active threats that need to be addressed at every phase of the development lifecycle.

## Security Challenges in AI Applications

Traditional software systems are mostly vulnerable to things like weak authentication, insecure code, and poor access controls. AI applications face those same problems, but they also open up entirely new attack surfaces that didn't exist before.

One of the most talked-about threats right now is **prompt injection**. In these attacks, a malicious user crafts an input that tricks a large language model into ignoring its original instructions or revealing information it shouldn't. This is especially dangerous in AI assistants and autonomous agents that have access to external tools, APIs, or databases — because manipulating the model's behavior can have real downstream consequences.

**Data poisoning** is another serious concern. AI models are only as good as the data they're trained on, and if an attacker can tamper with that training data, they can quietly influence how the model behaves. The result might be biased outputs, wrong predictions, or behavior the developers never intended.

There's also the growing risk of **model theft and extraction attacks**. Organizations pour significant resources into building proprietary AI models, and attackers have found ways to reconstruct those models by making large numbers of carefully crafted queries — essentially reverse-engineering a model without ever having direct access to it.

Finally, **sensitive information disclosure** remains an ongoing problem. When AI systems are trained on proprietary or personal data, they can inadvertently surface that information in their responses, sometimes in ways that are hard to predict or detect.

## Secure Development Practices

The good news is that many of these risks can be meaningfully reduced by integrating security into every phase of development — not just bolting it on at the end.

Strong **authentication and authorization** controls should be in place from the start. Only approved users and services should be able to interact with AI models and the infrastructure supporting them. Role-based access control is a practical way to limit what any single account can do, which reduces the blast radius if credentials are ever compromised.

**Input validation** deserves serious attention as well. Prompts and user inputs should be sanitized before they reach the model, which cuts down on the risk of successful prompt injection. Output filtering is equally important — it can prevent the model from returning sensitive or harmful content even when the input seems benign.

**Encryption** should be used to protect training datasets, model artifacts, and any data moving between services. Beyond that, organizations need comprehensive logging and monitoring in place. Being able to review what the AI system has been doing — and quickly spot unusual patterns — is essential for both detecting attacks and supporting any forensic work that follows.

One practice that's gaining traction specifically for AI is **adversarial testing and red-teaming**. These exercises involve deliberately probing deployed models to find weaknesses, simulating real attack scenarios to see how the system holds up under pressure. It's a level of testing that most traditional software teams aren't used to, but it's becoming a necessary part of the AI security toolkit.

For applications where the stakes are high — healthcare, financial services, critical infrastructure — **human oversight** should always remain part of the picture. Automated AI decisions in these domains need checkpoints where a human can review what's happening before actions are taken.

## AI Risk Management Frameworks

A few major organizations have stepped up to provide structured guidance for navigating these challenges.

The **National Institute of Standards and Technology (NIST)** released the AI Risk Management Framework (AI RMF) to help organizations identify, assess, and manage the risks that come with AI technologies. The framework is built around four core functions — Govern, Map, Measure, and Manage — that together push organizations to establish clear policies, evaluate risks systematically, watch how their systems perform over time, and keep improving their controls.

The **OWASP Foundation** has also put out targeted guidance through the OWASP Top 10 for LLM Applications. This list highlights the most pressing risks for teams building with large language models, including prompt injection, insecure output handling, training data poisoning, excessive model agency, and supply chain vulnerabilities. It's a practical starting point for any team deploying generative AI in their products.

## Conclusion

AI offers real, substantial benefits — but it also changes what software assurance professionals have to think about and protect against. Relying solely on traditional security techniques isn't enough when the system includes a machine learning model, sensitive training data, and components that can act autonomously based on what they generate.

By weaving secure development practices into the process from the beginning, running adversarial testing, enforcing strict access controls, and drawing on frameworks like the NIST AI RMF and OWASP's LLM guidance, organizations can build AI applications that hold up under real-world conditions. AI technology isn't slowing down, and neither is the need for engineers who know how to build it responsibly.

## References

1. National Institute of Standards and Technology. *AI Risk Management Framework (AI RMF 1.0).* January 2023.

2. National Institute of Standards and Technology. *AI Research – Security and Resilience.*

3. OWASP Foundation. *OWASP Top 10 for Large Language Model Applications.*

4. National Institute of Standards and Technology. *NIST Information Technology Laboratory AI Program.*
