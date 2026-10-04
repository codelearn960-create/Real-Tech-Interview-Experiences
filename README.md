# 🚀 Real Senior Java & Spring Boot Interview Experiences (2026 Edition)

> 💡 **PREPARING FOR SENIOR JAVA / SPRING BOOT ROLES (3 to 8+ YOE)?**
> 
> This GitHub repository contains **Chapter 1 (Airtel)** as a complete free preview.
> 
> Get the full **100+ Page Playbook** with real candidate debriefs, code review bug fixes, and system design scenarios for **Airtel, UKG, Deloitte, and IBS Software**:
> 
> 👉 **[CLICK HERE TO DOWNLOAD THE FULL PLAYBOOK ON TOPMATE](https://topmate.io/codelearn960/2332457)**

---

# 📘 CHAPTER 1: AIRTEL (BHARTI AIRTEL)

## 📌 Interview Snapshot

| Detail | Info |
|---|---|
| **Company** | Airtel (Bharti Airtel) |
| **Role** | Software Engineer (Java / Spring Boot) |
| **Total Rounds** | 2 |
| **Round 1 Focus** | Core Java, Code Review, Coding (2 Sum) |
| **Round 2 Focus** | Merge Sort, System Design, Code Review, Polymorphism |
| **Tools Mentioned** | IntelliJ IDEA, Java 17, Spring Boot 3.x, ABC DB |
| **Difficulty** | ⭐⭐⭐⭐ (Medium-Hard) |
| **Time per Round** | ~60–75 minutes |

---

## 🔵 ROUND 1 — Core Java, Code Review & Coding Fundamentals

### Q1. Tell me about yourself.

**Interview-Ready Answer:**
> "I am a backend software engineer with around 5 years of experience building microservices with Java, Spring Boot, and REST APIs. In my current role, I build and maintain core backend services handling transaction processing and vendor integrations. I specialize in writing clean code, optimizing database performance, and doing thorough code reviews."

---

### Q2. Walk me through your roles and responsibilities.

**Interview-Ready Answer:**
> "In my engineering team, my main responsibilities are:
> 1. **API Development:** Building RESTful microservices using Java 17 and Spring Boot.
> 2. **Testing:** Writing unit and integration tests using JUnit and Mockito.
> 3. **Code Reviews:** Reviewing peer pull requests for performance, thread safety, and clean architecture.
> 4. **Performance Tuning:** Fixing slow ABC DB queries and optimizing memory usage.
> 5. **Agile Sprint Delivery:** Collaborating in 2-week sprints with QA and DevOps teams."

---

### Q3. What are you currently working upon?

**Interview-Ready Answer:**
> "I am currently working on an Invoice Ingestion Service using Java 17, Spring Boot, ABC DB, and Kafka. A recent key task was optimizing our validation pipeline: I refactored in-memory list lookups to a `ConcurrentHashMap` and batched database writes, reducing processing latency from 800ms to 200ms."

---

### Q4. What happens after you code and unit test? Were you expecting code reviews?

**Interview-Ready Answer:**
> "After unit testing locally, I open a Pull Request. Peer code review is a mandatory quality gate in our process. Reviewers check logic, security, and thread safety. Simultaneously, our CI pipeline runs automated builds, unit tests, and SonarQube static analysis before code merges to staging."

---

### Q5. Code Review & Performance Optimization — TransactionService

**The Buggy Code Given in Interview:**
```java
@Service
public class TransactionService {
    private List<Transaction> transactions = new ArrayList<>();

    public Transaction findTransaction(String id) {
        for (Transaction transaction : transactions) {
            if (transaction.getId() == id) { // ❌ Bug 1: String comparison with ==
                return transaction;
            }
        }
        return null;
    }

    public List<Transaction> getTransactionsByCustomer(String customerId) {
        return transactions.stream()
                .filter(t -> t.getCustomerId().equals(customerId))
                .sorted((a, b) -> (int) (b.getAmount() - a.getAmount())) // ❌ Bug 2: Precision loss in sort
                .toList();
    }

    public void removeTransaction(String id) {
        for (Transaction transaction : transactions) {
            if (transaction.getId().equals(id)) {
                transactions.remove(transaction); // ❌ Bug 3: Throws ConcurrentModificationException
            }
        }
    }
}
```

**Bugs Found & Fixed:**
1. **String Comparison (`==` vs `.equals()`):** `==` compares object references instead of text values. Fixed with `.equals()`.
2. **`ConcurrentModificationException`:** Removing from `ArrayList` while iterating causes a runtime crash. Fixed by using a map.
3. **$O(N)$ Performance Bottleneck:** List traversal is slow. Fixed by using `ConcurrentHashMap` for $O(1)$ lookups.
4. **Precision Loss in Sort:** `(int) (b.getAmount() - a.getAmount())` truncates decimals. Fixed using `BigDecimal` comparisons.
5. **Money Handling:** Using `double` causes floating-point rounding errors. Fixed by using `BigDecimal`.

**Production-Ready Refactored Code:**
```java
@Service
public class TransactionService {
    private final Map<String, Transaction> transactionMap = new ConcurrentHashMap<>();

    public void addTransaction(Transaction transaction) {
        Objects.requireNonNull(transaction, "Transaction cannot be null");
        transactionMap.put(transaction.getId(), transaction);
    }

    public Optional<Transaction> findTransaction(String id) {
        return Optional.ofNullable(transactionMap.get(id));
    }

    public List<Transaction> getTransactionsByCustomer(String customerId) {
        return transactionMap.values().stream()
                .filter(t -> customerId.equals(t.getCustomerId()))
                .sorted((a, b) -> b.getAmount().compareTo(a.getAmount()))
                .toList();
    }

    public void removeTransaction(String id) {
        transactionMap.remove(id);
    }
}
```

---

### Q6. Can `main()` method be overridden? Why not?

**Interview-Ready Answer:**
> "No, `main()` cannot be overridden because it is declared `static`. Overriding relies on runtime polymorphism (dynamic binding) based on instance objects. Static methods are bound at compile time based on class references. Declaring `main()` in a subclass results in method hiding, not overriding."

---

### Q7. Two Sum — Optimal $O(N)$ Solution

```java
import java.util.*;

public class TwoSum {
    public static int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }
            map.put(nums[i], i);
        }
        return new int[] {};
    }

    public static void main(String[] args) {
        int[] result = twoSum(new int[]{2, 7, 11, 15}, 9);
        System.out.println("Output Indices: [" + result[0] + ", " + result[1] + "]");
    }
}
```

---

## 🔴 ROUND 2 — Merge Sort & System Design

### Q8. System Design — Handle 2M Customers, Accept & Process Orders in 3 Hours

**Interview-Ready Answer:**
> "To handle 2M orders in 3 hours (~185 req/sec average, with higher peaks), we decouple acceptance from processing:
> 1. **Ingestion:** Load Balancer (ALB) routes traffic across an Auto-scaling API Gateway group.
> 2. **Fast Acceptance:** Gateway publishes an `OrderCreatedEvent` to **Kafka** and immediately returns `HTTP 202 Accepted`.
> 3. **Processing Workers:** A consumer cluster reads Kafka partitions in parallel to process payments and update inventory.
> 4. **Database & Cache:** Primary ABC DB handles writes, Read Replicas handle queries, HikariCP manages connections, and Redis caches product data."

```
[2M Users] ──► [Load Balancer] ──► [API Gateway] ──► [Kafka Topic] ──► [Worker Pool] ──► [ABC DB]
                                                                             │
                                                                      [Redis Cache]
```

---

### Q9. Zero-Downtime Database Migration

**Interview-Ready Answer:**
> "I use the **Dual-Write Pattern**:
> 1. **Dual Write:** Update application code to write new data to both old and new DBs simultaneously.
> 2. **Backfill:** Run batch jobs to copy historical records from old DB to new DB.
> 3. **Verification:** Run reconciliation scripts to verify data consistency across databases.
> 4. **Canary Read Cutover:** Shift read traffic gradually to new DB via feature flags (5% -> 50% -> 100%).
> 5. **Cutover Writes:** Direct all writes to new DB, stop dual-writing, and archive the old DB."

---

> 🎯 **WANT THE COMPLETE 90+ PAGE PLAYBOOK?**
> 
> Get the full PDF containing complete debriefs and code solutions for **UKG, Deloitte, IBS Software, and Airtel**:
> 
> 👉 **[GET THE FULL PLAYBOOK HERE (Instant Download)](https://topmate.io/codelearn960/2332457)**
