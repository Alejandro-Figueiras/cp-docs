---
tags:
    - cp-algorithms
---

# Pruebas de Primalidad

Este artículo describe varios algoritmos para determinar si un número es primo o no.

## Probando divisiones

Por definición, un número primo no tiene más divisores que $1$ y él mismo.
Un número compuesto tiene al menos un divisor adicional, llamémoslo $d$.
Naturalmente $\frac{n}{d}$ también es un divisor de $n$.
Es fácil ver que, o bien $d \le \sqrt{n}$, o bien $\frac{n}{d} \le \sqrt{n}$, por lo tanto uno de los divisores $d$ y $\frac{n}{d}$ es $\le \sqrt{n}$.
Podemos utilizar esta información para comprobar la primalidad.

Tratamos de encontrar un divisor no trivial, comprobando si alguno de los números entre $$2$$ y $$\sqrt{n}$$ es divisor de $n$.
Si es un divisor, entonces $n$ definitivamente no es primo, de lo contrario lo es.

```cpp
bool isPrime(int x) {
    for (int d = 2; d * d <= x; d++) {
        if (x % d == 0)
            return false;
    }
    return x >= 2;
}
```

Esta es la forma más simple de una comprobación de primos.
Se puede optimizar bastante esta función, por ejemplo, comprobando sólo todos los números impares en el bucle, ya que el único número primo par es 2.
Múltiples optimizaciones de este tipo se describen en el artículo sobre **factorización de enteros**
<!-- factorización de enteros (factorization.md). -->

## Prueba de Primalidad de Fermat

Se trata de una prueba probabilística.

<!-- (véase también función totiente de Euler(phi-function.md)) -->
El pequeño teorema de Fermat <!-- here --> establece que para un número primo $p$ y un número entero coprimo $a$ se cumple la siguiente ecuación:

$$
a^{p-1} \equiv 1 \bmod p
$$

En general, este teorema no se aplica a los números compuestos.

Se puede utilizar para crear una prueba de primalidad.
Elegimos un número entero $2 \le a \le p - 2$, y comprobar si la ecuación se mantiene o no.
Si no se cumple, por ejemplo, $a^{p-1} \not\equiv 1 \bmod p$, sabemos que $p$ no puede ser un número primo.
En este caso llamamos a la base $a$ un *testigo de Fermat* para la compositividad de $p$.

Sin embargo, también es posible que la ecuación se cumpla para un número compuesto.
Así que si la ecuación se cumple, no tenemos una prueba de la primalidad.
Sólo podemos decir que $p$ es *probablemente primo*.
Si resulta que el número es realmente compuesto, llamamos a la base $a$ un *mentiroso de Fermat*.

Realizando la prueba para todas las bases posibles $a$, podemos demostrar que un número es primo.
Sin embargo, esto no se hace en la práctica, ya que supone mucho más esfuerzo que hacer simplemente la *división de prueba*.
En su lugar, la prueba se repetirá varias veces con elecciones aleatorias para $a$.
Si no encontramos ningún testigo de la compositividad, es muy probable que el número sea primo.

```cpp
bool probablyPrimeFermat(int n, int iter=5) {
    if (n < 4)
        return n == 2 || n == 3;

    for (int i = 0; i < iter; i++) {
        int a = 2 + rand() % (n - 3);
        if (binpower(a, n - 1, n) != 1)
            return false;
    }
    return true;
}
```

Usamos [Exponenciación Binaria](../fundamentals/binary-exp.md) para calcular eficientemente la potencia $a^{p-1}$.

Sin embargo, hay una mala noticia:
existen algunos números compuestos donde $a^{n-1} \equiv 1 \bmod n$ se mantiene para todos los $a$ coprimo a $n$, por ejemplo para el número $561 = 3 \cdot 11 \cdot 17$.
Estos números se llaman *números de Carmichael*.
La prueba de primalidad de Fermat sólo puede identificar estos números si tenemos mucha suerte y elegimos una base $a$ con $\gcd(a, n) \ne 1$.

La prueba de Fermat se sigue utilizando en la práctica, ya que es muy rápida y los números Carmichael son muy raros.
Por ejemplo, sólo existen 646 números de este tipo por debajo de $10^9$.

<!-- ## Miller-Rabin primality test

The Miller-Rabin test extends the ideas from the Fermat test.

For an odd number $n$, $n-1$ is even and we can factor out all powers of 2.
We can write:

$$n - 1 = 2^s \cdot d,~\text{with}~d~\text{odd}.$$

This allows us to factorize the equation of Fermat's little theorem:

$$\begin{array}{rl}
a^{n-1} \equiv 1 \bmod n &\Longleftrightarrow a^{2^s d} - 1 \equiv 0 \bmod n \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-1} d} - 1) \equiv 0 \bmod n \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-2} d} + 1) (a^{2^{s-2} d} - 1) \equiv 0 \bmod n \\\\
&\quad\vdots \\\\
&\Longleftrightarrow (a^{2^{s-1} d} + 1) (a^{2^{s-2} d} + 1) \cdots (a^{d} + 1) (a^{d} - 1) \equiv 0 \bmod n \\\\
\end{array}$$

If $n$ is prime, then $n$ has to divide one of these factors.
And in the Miller-Rabin primality test we check exactly that statement, which is a more stricter version of the statement of the Fermat test.
For a base $2 \le a \le n-2$ we check if either

$$a^d \equiv 1 \bmod n$$

holds or

$$a^{2^r d} \equiv -1 \bmod n$$

holds for some $0 \le r \le s - 1$.

If we found a base $a$ which doesn't satisfy any of the above equalities, then we found a *witness* for the compositeness of $n$.
In this case we have proven that $n$ is not a prime number.

Similar to the Fermat test, it is also possible that the set of equations is satisfied for a composite number.
In that case the base $a$ is called a *strong liar*.
If a base $a$ satisfies the equations (one of them), $n$ is only *strong probable prime*.
However, there are no numbers like the Carmichael numbers, where all non-trivial bases lie.
In fact it is possible to show, that at most $\frac{1}{4}$ of the bases can be strong liars.
If $n$ is composite, we have a probability of $\ge 75\%$ that a random base will tell us that it is composite.
By doing multiple iterations, choosing different random bases, we can tell with very high probability if the number is truly prime or if it is composite.

Here is an implementation for 64 bit integer.

```cpp
using u64 = uint64_t;
using u128 = __uint128_t;

u64 binpower(u64 base, u64 e, u64 mod) {
    u64 result = 1;
    base %= mod;
    while (e) {
        if (e & 1)
            result = (u128)result * base % mod;
        base = (u128)base * base % mod;
        e >>= 1;
    }
    return result;
}

bool check_composite(u64 n, u64 a, u64 d, int s) {
    u64 x = binpower(a, d, n);
    if (x == 1 || x == n - 1)
        return false;
    for (int r = 1; r < s; r++) {
        x = (u128)x * x % n;
        if (x == n - 1)
            return false;
    }
    return true;
};

bool MillerRabin(u64 n, int iter=5) { // returns true if n is probably prime, else returns false.
    if (n < 4)
        return n == 2 || n == 3;

    int s = 0;
    u64 d = n - 1;
    while ((d & 1) == 0) {
        d >>= 1;
        s++;
    }

    for (int i = 0; i < iter; i++) {
        int a = 2 + rand() % (n - 3);
        if (check_composite(n, a, d, s))
            return false;
    }
    return true;
}
```

Before the Miller-Rabin test you can test additionally if one of the first few prime numbers is a divisor.
This can speed up the test by a lot, since most composite numbers have very small prime divisors.
E.g. $88\%$ of all numbers have a prime factor smaller than $100$.

### Deterministic version

Miller showed that it is possible to make the algorithm deterministic by only checking all bases $\le O((\ln n)^2)$.
Bach later gave a concrete bound, it is only necessary to test all bases $a \le 2 \ln(n)^2$.

This is still a pretty large number of bases.
So people have invested quite a lot of computation power into finding lower bounds.
It turns out, for testing a 32 bit integer it is only necessary to check the first 4 prime bases: 2, 3, 5 and 7.
The smallest composite number that fails this test is $3,215,031,751 = 151 \cdot 751 \cdot 28351$.
And for testing 64 bit integer it is enough to check the first 12 prime bases: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, and 37.

This results in the following deterministic implementation:

```cpp
bool MillerRabin(u64 n) { // returns true if n is prime, else returns false.
    if (n < 2)
        return false;

    int r = 0;
    u64 d = n - 1;
    while ((d & 1) == 0) {
        d >>= 1;
        r++;
    }

    for (int a : {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}) {
        if (n == a)
            return true;
        if (check_composite(n, a, d, r))
            return false;
    }
    return true;
}
```

It's also possible to do the check with only 7 bases: 2, 325, 9375, 28178, 450775, 9780504 and 1795265022.
However, since these numbers (except 2) are not prime, you need to check additionally if the number you are checking is equal to any prime divisor of those bases: 2, 3, 5, 13, 19, 73, 193, 407521, 299210837. -->

## Problemas de Práctica

- [SPOJ - Prime or Not](https://www.spoj.com/problems/PON/)
- [Project euler - Investigating a Prime Pattern](https://projecteuler.net/problem=146)