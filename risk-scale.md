## Risk Scale Evaluation Framework

Every scenario described in the project is associated the a risk scale.

The risk scale is designed to evaluate security threats based on three key aspects: **Complexity**, **Stealth**, and **Leakage Amount**. Each aspect is assessed on a scale of **High**, **Medium**, or **Low**.

The risk scale is very simple (nearly simplisitic) but it is done on purpose. More complexy impact scale exists like the percing index the [Piercing index](https://github.com/piercing-index/cloud-vulnerabilities) must is far mor complex to evaluate and "feel" excalty what is hide behind the index.

### 1. Complexity (Complexity to Execute the Risk Scenario)

- **High Complexity**
  - Execution requires advanced skills, significant resources, specialized knowledge, or access.
  - Involves sophisticated techniques, insider knowledge, or exploiting less-known vulnerabilities.

- **Medium Complexity**
  - Requires a moderate level of skill or resources.
  - Involves using publicly known vulnerabilities, standard hacking tools, or methods well-documented in the public domain.

- **Low Complexity**
  - Can be executed with minimal skills or resources.
  - Involves exploiting widely known and easily accessible vulnerabilities, basic social engineering techniques, or other low-effort methods.

### 2. Stealth (To What Extent the Risk Scenario Can be Detected)

- **High Stealth**
  - Very difficult to detect.
  - Involves advanced methods to conceal activities, leaving minimal traces.
  - Could include sophisticated malware, advanced persistent threats (APTs), or covert channels.

- **Medium Stealth**
  - Some level of detectability, but not easily or immediately apparent.
  - Methods generate some noise or leave traces, but can evade basic or standard security measures.

- **Low Stealth**
  - Easily detectable.
  - Methods are obvious and likely to trigger security alerts or be noticed by users or administrators.
  - Includes noisy network scans, easily spotted malware, or obvious phishing attempts.

### 3. Leakage Amount (What Amount of Data Can Be Leaked in a Data Leakage Scenario)

- **High Leakage Amount**
  - Large volume of sensitive data can be exposed.
  - Involves full database dumps, access to numerous confidential files, or extensive access to sensitive personal data.

- **Medium Leakage Amount**
  - Moderate amount of data is at risk.
  - Involves access to a limited set of sensitive documents, partial database access, or exposure of some personal data.

- **Low Leakage Amount**
  - Minimal amount of data can be leaked.
  - Involves exposure of non-sensitive or public data, or very limited access to confidential information.
