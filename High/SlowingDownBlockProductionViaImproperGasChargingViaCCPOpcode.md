---

## **2. High Severity: Slowing Down Block Production via Improper Gas Charging for `CCP` Opcode**

### **Concept**
This finding relates to a **gas mispricing vulnerability** in the implementation of a `code-copy` opcode (`CCP`). The issue can be exploited to perform a Denial-of-Service (DoS) attack by slowing down block production.

### **How the Exploit Works**
- The `CCP` opcode is used to copy a specified number (`n`) of bytes from a contract’s bytecode into memory.
- If `n` is larger than the available bytecode length, the opcode **fills the remaining space with zeros**, similar to the `memory-clear` opcode (`MCL`).
- The vulnerability: **Gas fees are only charged based on the size of the contract’s bytecode**, **not** the actual number of bytes copied into memory.

### **Exploitation**
1. **Choose a small contract** (e.g., a contract with 1 byte of bytecode).
2. Call `CCP` with `n` set to the size of the entire memory area (much larger than the contract's actual size).
   - The opcode clears a large area of memory but only charges gas based on the tiny contract size (e.g., 15 gas instead of the expected 20,000 gas).
3. By looping this operation:
   - The node ends up performing extensive memory clearing operations at a very low gas cost.
   - This significantly increases block production time, leading to delayed transactions and network congestion.

### **Impact**
- The Proof-of-Concept (PoC) provided by the user showed **a 24x slowdown** in block production (target block time was 1 second, but PoC took 24 seconds).
- The exploit is a classic example of a **DoS attack via resource exhaustion**, where the node gets overloaded with expensive, underpriced operations.
- In a production setting, this could lead to severe degradation of the network's performance and availability.

### **Why This Bug Was High Severity**
- **Denial of Service**: The attack slows down block production, affecting the entire network's throughput and stability.
- **Gas Accounting Flaw**: Incorrect gas pricing is a critical issue in any blockchain, as it allows attackers to perform costly operations at low cost, disrupting fair resource usage.
- **Potential for Further Optimization**: The user noted that the PoC could be further optimized (e.g., using raw assembly), potentially leading to even greater disruption.

---

## **Takeaways**

1. **Cross-Layer Security Issues**: The critical finding highlights the inherent risks in cross-chain systems where state discrepancies can lead to severe vulnerabilities.
2. **Gas Pricing and DoS Risks**: The high-severity finding underscores the importance of accurate gas pricing for opcodes to prevent resource-based attacks.
3. **Advanced Exploitation Techniques**: Both findings required a high level of expertise to identify and exploit, especially the customized testing setup for the bridge vulnerability.

In conclusion, the findings demonstrate a deep understanding of blockchain internals, cross-chain mechanisms, and low-level opcode behavior, showcasing the complexity and subtlety of high-impact security vulnerabilities in modern blockchain architectures.
