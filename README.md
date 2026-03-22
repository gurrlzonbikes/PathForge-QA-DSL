# 🧭 PathForge — A Coverage DSL

A domain-specific language (DSL) for expressing, scaling, and **comparing test coverage** through explicit system traversal paths.

---

## 🌱 The Idea

Traditional QA describes **user behavior**.

PathForge describes:

> what the system actually executes

---

## 🔤 The DSL


`RD → TB → TB → PB → CD`


Example:


`RD1 → TB2 → TB13 → TB4 → PB1 → CD1`


Each path encodes:

- **RD** → execution context  
- **TB** → technical components exercised  
- **PB** → parameter variation  
- **CD** → external interactions  

---

## 🎯 Why It Matters

### ❌ Traditional QA
- Coverage is implicit  
- Scenarios duplicate the same execution paths  
- More tests ≠ more coverage  

### ✅ PathForge
- Coverage is **explicit and traceable**  
- Paths map directly to system structure  
- Gaps and overlaps are visible  

> You don’t guess coverage — you see it.

---

## 🚨 Built for Non-Regression

PathForge ensures tests:

- traverse shared system layers  
- exercise dependencies  
- reach integration points  

Result:

- fewer scenarios  
- higher coverage density  
- stronger confidence  

---

## 🌿 Parameterization Without Noise

Instead of rewriting tests:


Same path + different PB → expanded coverage


- No duplication  
- Stable intent  
- Controlled variation  

---

## 🤖 AI-Ready by Design

Traversal paths act as a **stable intermediate representation**.

- AI generates **structured paths**, not raw test cases  
- Coverage remains **traceable and explainable**  
- Variants expand coverage without duplication  

---

## 📊 Campaigns as Data

A test campaign becomes a **DSL index table**:


```
P01 TB2.b → TB1.a/b → TB4.a–i → PB1[Arbitrum] → CD1[MetaMask]
P02 TB2.a/c → TB4.c → PB1[Optimism] → CD1[Ledger Live]
```


Each row = one **coverage path**

---

## 🔍 Diffable Coverage

Because paths are structured:

- campaigns can be compared over time  
- coverage evolution becomes visible  

You can detect:

- added coverage  
- removed coverage  
- modified execution paths  

> Coverage becomes versionable.

## 📊 Example — Campaign Diff

### 🧾 Campaign A


```
P01 TB2 → TB4 → PB1[Arbitrum] → CD1[MetaMask]
P02 TB5 → TB6 → CD1[Ledger Live]
P03 TB8 → PB3[DAI] → CD2[Aave]
```


---

### 🧾 Campaign B


```
P01 TB2 → TB4 → PB1[Arbitrum] → CD1[MetaMask]
P02 TB5 → TB7 → CD1[Ledger Live]
P04 TB10 → PB3[USDC] → CD2[Uniswap]
```


---

### 🔍 Diff Analysis

#### ➕ Added Coverage
`P04 TB10 → PB3[USDC] → CD2[Uniswap]`

New system path introduced:
- new technical branch (**TB10**)  
- new integration (**Uniswap**)  

---

#### ➖ Removed Coverage
`P03 TB8 → PB3[DAI] → CD2[Aave]`

Lost coverage:
- Aave integration  
- DAI-related flows  

⚠️ Potential regression risk

---

#### 🔁 Modified Paths

P02:

```
TB5 → TB6 → CD1
TB5 → TB7 → CD1
```

Change in execution:
- TB6 replaced by TB7  
- altered system behavior coverage  

---

---

## 🧠 Key Shift

| Traditional QA | PathForge |
|------|--------|
| Steps describe behavior | Paths describe execution |
| Coverage is implicit | Coverage is explicit |
| Test cases multiply | Paths are recomposed |
| Hard to compare | Fully diffable |

---

## 🌱 Summary

PathForge turns testing into a:

- structured  
- composable  
- **diffable** system  

> Quality is encoded in traversal — not lost in test cases.
