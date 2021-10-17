# Assignment LMC Programming 04-11-2021

# Algorithm

**Key marking points:**
- Marks are awarded for *Robustness*.
- Marks are awarded for *Correctness*.
- Marks are awarded for *Efficiency*.
	- Measured in number of *Fetch-Execute-Cycles*.
- ==No marks are awarded for *Space Efficiency*.==

**Requirements:**
- Takes 3 inputs, $\large a$, $\large b$, $\large x$.
- Calculates $\large a + bx + x^{2}$.
- In the event of overflow output $\large 999$.

## Naive algorithm
1. Take ($\large a$, $\large b$, $\large x$) Inputs and store them.
2. Calculate $\large bx$ using *repeated addition*.
	1. Store a copy of $\large x$ as $\large i$.
	2. Add $\large x$ to $\large b$ and decrement $\large i$
	3. Check for overflow of $\large b$ by seeing if result is $\large < 0$.
		1. If an overflow occures, output $\large 999$ and halt.
	4. When $\large i$ is $\large 0$, exit loop, otherwise repeat from (1).
3. Add $\large b$ (which should now hold the result $\large bx$) to $\large a$.
4. Check for overflow of $\large a$ by seeing if result is $\large < 0$. 
	1.  If an overflow occures, output $\large 999$ and halt.
5.  Calculate $\large x \times x$ using *repeated addition* (as described above).
6.  Add $\large x$ (which should now hold the result $\large x \times x$) to $\large a$.
7.  Check for overflow of $\large a$ by seeing if result is $\large < 0$. 
	1.  If an overflow occures, output $\large 999$ and halt.
8.  Output $\large a$ and halt.

## Optimizing the naive algorithm
### Addressing the use of "functions"
As this assignment is efficiency based on the number of *Fetch-Execute-Cycles*, it is counter-intuitive to use functions as these have overhead costing a number of *Fetch-Execute-Cycles* to branch to. Thus functions should be avoided.

### Optimizing multiplication:
#### General Optimization
The naive multiplication algorithm for two values, $\large a$ and $\large b$ is described as:
1. Store a copy of $\large a$ as $\large i$.
2. Add $\large a$ to $\large b$ and decrement $\large i$
3. Check for overflow of $\large b$ by seeing if result is $\large < 0$.
	1. If an overflow occures, output $\large 999$ and halt.
4. When $\large i$ is $\large 0$, exit loop, otherwise repeat from (1).

The problem is that the amount of looping is dependent on just $\large a$. If $\large b < a$ then we are looping more than we need to thus using more *Fetch-Execute-Cycles*. Instead we should:
1. If $\large b < a$ go to (6).
2. Store a copy of $\large a$ as $\large i$.
3. Add $\large a$ to $\large b$ and decrement $\large i$
4. Check for overflow of $\large b$ by seeing if result is $\large < 0$.
	1. If an overflow occures, output $\large 999$ and halt.
5. When $\large i$ is $\large 0$, go to (10), otherwise repeat from (1).
6. Store a copy of $\large b$ as $\large i$.
7. Add $\large b$ to $\large a$ and decrement $\large i$
8. Check for overflow of $\large a$ by seeing if result is $\large < 0$.
	1. If an overflow occures, output $\large 999$ and halt.
9. When $\large i$ is $\large 0$, go to (10), otherwise repeat from (1).
10. Exit loop.

This is only more efficient if the number of *Fetch-Execute-Cycles* added to perform the comparison $\large b < a$ is less than the *Fetch-Execute-Cycles* saved from this algorithm, which is only the case when $\large a$ and $\large b$ differ dramatically.
	- $\large a = 10, b = 11$ vs $\large a = 3, b = 78$.
	
#### Re-using Values
Another optimization we can do is to use the result of any part of our multiplication to aid in another.

For example:
- When calculating $\large x \times x$ we can use that progress to calculate $\large bx$:
	- Case 1 $\large b == x$:
		- Simple the result can be used for $\large bx$.
	- Case 2 $\large b < x$:
		- When performing *repeated addition* for $\large x \times x$ the result for $\large bx$ will appear midway:
			- E.g:
		$\large x = 4$
		$\large b = 2$
		$\large x \times x = (4 + 4) + 4 + 4$
		$\large x \times b = (4 + 4)$
		
		- Therefore, midway *repeat addition* we can account for this and take the output midway for $\large bx$.
	- Case 3 $\large x < b$:
		- When performing *repeated addition* for $\large x \times x$ the result for $\large bx$ can be given a headstart:
			- E.g:
		$\large x = 2$
		$\large b = 4$
		$\large x \times x = (4 + 4)$
		$\large x \times b = (4 + 4) + 4 + 4$
		
		- Therefore, after performing $\large x \times x$ we can continue where we left off to calculate $\large bx$.

This can be taken further by accounting for **Case 2** and **Case 3**. This is because **Case 3** will run in less *Fetch-Execute-Cycles* than **Case 2** as we do not need to perform a midway check as we perform *repeated addition*. Because of this we can simply change the order of when we perform $\large x \times x$ and $\large bx$ such that we always are in **Case 3**:
- In the event that $\large x > b$  we perform $\large bx$ first:
	$\large x = 4$
	$\large b = 2$
	$\large b \times x = (4 + 4)$
	$\large x \times x = (4 + 4) + 4 + 4$
- In the event that $\large x < b$  we perform $\large x \times x$ first:
	$\large x = 2$
	$\large b = 4$
	$\large x \times x = (2 + 2)$
	$\large b \times x = (2 + 2) + 2 + 2$
- In the event that $\large x = b$ then we can re-use our result from one calculation for the other.

#### Merging Optimization Ideas

To those that are keen eye'd you will notice that *Re-using Values* conflicts with the *General Optimization*. As the swap used to ensure we are looping as little as possible makes performing this algorithm harder:
- In the event that $\large x > b$  we perform $\large bx$ first:
	$\large x = 8$
	$\large b = 2$
	$\large b \times x = (8 + 8)$
	$\large x \times x = (8 + 8) + 8 + 8 + 8 + 8 + 8 + 8$
- In the event that $\large x < b$  we perform $\large x \times x$ first:
	$\large x = 2$
	$\large b = 8$
	$\large x \times x = 2 + 2$
	$\large b \times x = 8 + 8$
- In the event that $\large x = b$ then we can re-use our result from one calculation for the other.

As you can see in the $\large x < b$ case, $\large bx$ was optimized by *General Optimization*, but in the $\large x > b$ case we can assist *General Optimization* further using this algorithm. 
- This means that we use *General Optimization* for the $\large x < b$ case but we only use *Re-using Values* for the $\large x > b$ and $\large x = b$ cases.

We can take this further as both *General Optimization* and *Re-using Values* require a comparison of $\large x > b$ or $\large x = b$ etc... of some kind which can be reused for both algorithms so we are not adding extra *Fetch-Execute-Cycles* performing more comparisons by merging the two methods.

### Early Stopping
Another way we can optimize is by *early stopping* when we know an overflow will occure.
For example, when we obtain the $\large x$ input we can check if $\large x < 32$ as if $\large x \geq 32$ then we know for sure that the $\large x^{2}$ operation will overflow and we can halt early.

### Not performing unnecessary checks / calculations
We can also infer some facts about the output based on the input. For example:
- If $\large x = 0$ then we know that our output can never overflow and that it will be $\large a$.
- If $\large b = 0$ then we can skip calculating $\large bx$.
- If $\large a = 0$ then we can skip adding  $\large a$.
	- This final statement, however is not worth implementing as the *Fetch-Execute-Cycles* added to perform the check is $\large \geq$ than the saved *Fetch-Execute-Cycles* (which is just 1 add operation).

# Assignment Document
![[assignmentCSys2021.pdf]]
