
# Dry Ice ’n Co — CTF Write-up (Partial Solve)

## Challenge Summary

This challenge presents a small Spring Boot e-commerce web application for buying dry ice products — and a secret product named `"flag"`.  
The goal is to unlock the flag by purchasing it, despite its extremely high cost and restrictions.

## Application Overview

- Language & stack: Java, Spring Boot, Thymeleaf.
- Products: Stored in memory, with name, description, price, and stock.
- Cart & balance: Per session (`HttpSession`).
- Coupon: One valid coupon `SMILEICE` for 20% off.
- Admin panel: Can add new products.

## Key Code Details

`/admin/add-product`

```java
if ((user.admin = true) && user != null && name != "flag")
```

This uses an assignment instead of a comparison, so any user can become admin and add new products.

`/purchase`

```java
if (cart.canAfford()) {
    boolean allInStock = ...
    if (allInStock) {
        reduce stock...
        if (items.size() == 1 && items.get(0).getName().equals("flag")) {
            boughtFlag = true;
        }
        cart.purchase();
    }
}
```

This ensures that the flag is unlocked only if the cart has exactly one item named `"flag"` and the total can be afforded.

## Vulnerabilities Discovered

| ID                      | Description                                    | Impact |
|----                     |--------------                                  |--------|
| 1. Assignment in `if`   | `(user.admin = true)` instead of comparison    | Any user can become admin |
| 2. Reference comparison | `name != "flag"` checks reference, not content | Weak duplicate prevention |
| 3. No stock locking     | Global in-memory stock                         | Possible oversell race |
| 4. Empty cart logic     | Purchase works with empty cart                 | Harmless |

## What I Tried

| Attempt                     | Idea                                      | Result |
|---------                    |------                                     |--------|
| Privilege escalation        | POST to `/admin/add-product`              | Worked |
| Overwrite real `"flag"`     | Added a cheaper `"flag"`                  | Ignored: list always picks first match |
| Homoglyphs & encoding       | Used Unicode, null bytes, double encoding | Displayed, but `.equals("flag")` passes only for exact match |
| Negative price              | Created product with negative price       | Neutralized by `Math.abs` |
| Intercepted purchase        | Paused `/purchase` and modified cart      | Server uses session snapshot, so forging body doesn’t work |
| Repeated purchase           | Replay `/purchase` multiple times         | Cart is emptied after first |

## How I Used Burp Suite

- Intercepted requests: adding, removing, applying coupon, purchasing.
- Crafted custom POST requests to test admin bug and fake `"flag"` products.
- Intercepted `/purchase` to test cart manipulation.
- Verified that the server always picks the first `"flag"` in the product list.
- Applied the coupon to observe how the total changes.

## Why a Cheap `"flag"` Fails

When adding to the cart:

```java
availableProducts.stream()
    .filter(p -> p.getName().equals(productName))
    .findFirst()
```

This always picks the first product named `"flag"`. So adding a fake cheap `"flag"` works in the list but is never chosen for cart or purchase.


## Learning Notes — Docker Deployment Basics

Before testing exploits, I learned how to build and deploy the app locally with Docker for safe testing.

What I did:

- Build with Gradle:
  ```bash
  ./gradlew clean bootJar
  ```

- Build Docker image:
  ```bash
  docker build -t dryice .
  ```

- Run the app in a container:
  ```bash
  docker run -p 8080:8080 dryice
  ```

- Troubleshoot:
  - Resolved port conflicts.
  - Rebuilt and redeployed when changing the app.
  - Used Docker logs to check runtime issues.

Running it in Docker allowed me to:
- Isolate tests.
- Safely experiment with Burp Suite intercepts.
- Replay real traffic and verify exploits.

## Conclusion
During the CTF, I was able to identify multiple security flaws in the application, including privilege escalation through an assignment bug, unsafe string comparison, insecure stock handling, and poor request validation. I used Burp Suite extensively to intercept, modify, and replay requests, and I verified these issues on a local Docker deployment I set up myself.

Despite finding these vulnerabilities and testing several creative bypass techniques — including creating fake flag products, using Unicode tricks, manipulating requests, racing purchases, and testing negative prices — I was unable to solve the challenge fully during the CTF time limit.

What I missed:
The key intended solution was a classic Java-specific bug: the total cart price uses a 32-bit signed integer (int), which can overflow. By adding a maximum integer value (2,147,483,647) and another small number, the total cost calculation overflows into a negative value. Because Java’s Math.abs function cannot fix Integer.MIN_VALUE properly, the negative total passes through the cost check. This means balance >= total is always true for a negative total, so the app mistakenly believes the user can afford the expensive flag.

By combining this overflow with the valid coupon code (which multiplies the negative total again, ensuring the sign or magnitude remains favorable), it becomes possible to gain money instead of spending it — bypassing the cost check and allowing a purchase of the real flag.

Lesson Learned:
This was a classic example of why signed integer limits must be handled carefully in secure applications, especially when using Math.abs on signed limits. I learned how subtle language-specific quirks can break business logic and lead to serious financial bypass vulnerabilities.


