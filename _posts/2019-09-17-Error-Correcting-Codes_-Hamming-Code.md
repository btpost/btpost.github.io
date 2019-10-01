---
title: Error Correcting Codes- Hamming Code
topic: Math
---
Error correcting codes are not only incredibly interesting, but also incredibly useful. They can be found in various forms in all digital communication. There are a wide variety, with one of the most common being the 16-bit Cyclic Redundancy Check found in TCP/IP, to some of the more complex CRCs used in modern Ethernet frames, as well as wireless communications. These provide a layer of error detection to help filter the transmission of corrupted data. Practical and easy to implement, these codes help us transmit data quickly and safely. But, they aren't *perfect*.

Hamming code, however, is what's known as a perfect code. In order to fully explain what this means, I will have to present some definitions.

So first, what is a code, mathematically speaking. A code is a function that maps a set of possible messages, call it set $$M$$, to a set of codewords, call that set $$C$$. When dealing with error-correcting codes, the elements of $$M$$ will generally be shorter than the elements of $$C$$. This is not always the case, for example [Gray Code](https://en.wikipedia.org/wiki/Gray_code), can assist in error detection without increasing the size of transmission. But, if we want a perfect code, some redundancy will be necessary. The length of the codewords found in $$C$$ is known as the code's *block length*. 

Now, each code possess more special characteristics we can work with to measure it's efficiency. Let's look at a simple parity code of *block length* $$n = 8$$ on the alphabet $$\Sigma^n = {0,1}^8$$, with 7 information bits. This means codewords will all be of length 8, with the original messages being length seven. A simple way to check a code's efficiency is to simply compare it's block length to the number of information btis. This is known as its *rate*. For the parity code above, which I will be writing as $$C_{XOR}$$, we find the rate $$r = 7/8$$. We can also look at the code's *distance*, which is the minimum number of differences between each codeword. When encoding into $$C_{XOR}$$, we use the following mapping of binary vectors of size 7, given by $$\bar x$$, to binary vectors of size 8:
<p style="text-align: center;"> $$E(\bar x) = (x_1, x_2, ..., x_7, x_1 \mathbf{XOR} x_2 \mathbf{XOR} ... x_7 )$$ </p>

So, our *encoding function* $$E$$ above, returns each of the seven information bits, followed by these 7 bits $$\mathbf{XOR}$$'d with eachother. We can now note that any message with an even number of 1's will have a zero appended to it, while messages with an odd number of zeros will recieve a 1 at the end. This means that $$C_{XOR}$$ has the distance $$d = 2$$. It would be impossible to change a single information bit without also changing the parity bit, and vice-versa.

The notion of distance is one of the main drivers of error-correcting codes. We can see here that $$C_{XOR}$$ only includes half of the possible vectors in $${0,1}^8$$. Meaning if there is a single bit error in a recieved message, we will know that it's incorrect. Unfortunately, since this code is has distance $$d=2$$, there will be two codewords that are distance 1 from an erroneous message, so it would be impossible to determine which one was intended. Further, if there was more than one error, we may not even be able to tell an error occurred. So, $$C_{XOR}$$ is called, *1-error detecting*. For any code of distance $$d$$, it is possible to detect up to $$d-1$$ errors. 

Now we will take a look at a famous *error-correcting* code, namely Hamming (7,4). Hamming (7,4) has a block length of 7, with 4 information bits. A perfect code can be described as follows: Let $$d = 2e+1$$ with $$d$$ being distance and $$e$$ being the number of possible errors. Then, for any received message containing $$e$$ errors, there is only one codeword with $$e$$ different bits. Hamming (7,4) has a distance $$d=3$$, so if there's only one error in a recieved message, there's only one possible codeword it could be. This makes Hamming (7,4) *1-error-correcting*, since it can decode messages with one error to correct codewords.

What does this look like in practice? Well, first we have to make a code that satisfies these qualities. There are many ways to construct binary codes, but I think the simplest it to present a linearly independent set of vectors that span the code, which are distance 3 apart. Knowing that we are sending 4 information bits, we can start with the identity matrix:

<p style="text-align: center;">$$ {\begin{array}{cccc} 1 & 0 & 0 & 0 \\
															  0 & 1 & 0 & 0 \\
															  0 & 0 & 1 & 0 \\
															  0 & 0 & 0 & 1 \\
															  \end{array} 
															  $$</p>

Now, we can see the identity matrix is linearly independent, but each vector is only distance 1 from any other. Hamming (7,4) requires distance 3, so we need to make use of our parity bits to rectify this. Since we have distance one, we need a set of 4 3-bit strings that have at least distance 2. We can use the following four:

<p style="text-align: center;">$$ {\begin{array}{ccc} 1 & 1 & 0 \\
															  1 & 0 & 1 \\
															  0 & 1 & 1 \\
															  1 & 1 & 1 \\
															  \end{array}
															  $$</p>

Now, if we simply append these directly to the parity matrix, we end of with a basis for our code, with 4 binary strings of length 7. 

<p style="text-align: center;">$$ {\begin{array}{cccc} 1 & 0 & 0 & 0 & 1 & 1 & 0 \\
															  0 & 1 & 0 & 0 & 1 & 0 & 1 \\
															  0 & 0 & 1 & 0 & 0 & 1 & 1 \\
															  0 & 0 & 0 & 1 & 1 & 1 & 1 \\
															  \end{array}
															  $$</p>

This is what's known as a generator matrix. Every possible codeword in this code is a linear combination of these four strings. Note that this is the standard generator matrix for Hamming (7,4), but we could make many codes with the exact behavior but different generators (24 I think, but I don't have a perfect proof). Now, we can define the parity checks for each of the parity bits. Each parity bit $$\mathbf{XOR}$$'s three information bits. Knowing this, we can see by inspecting the matrix that they're defined as follows:

Parity Bit 1 (bit 5): $$x_1 \mathbf{XOR} x_2 \mathbf{XOR} x_4$$
Parity Bit 2 (bit 6): $$x_1 \mathbf{XOR} x_3 \mathbf{XOR} x_4$$
Parity Bit 3 (bit 7): $$x_2 \mathbf{XOR} x_3 \mathbf{XOR} x_4$$

I'm not perfectly clear on the mathematical justification for the arrangement of the parity bits. If you can explain it more clearly, please feel free to email me.

Having all this set up, we can start to do some decoding. Say we get the following message with one error transmitted to us: 1000 011.

First, we can see that the parity bits are off. If our message was intended to be 1000, then we would need to have parity 110 (as we can see in our basis above). Then how do we determine the bit with the error? By inspection, we can see that we would need to change two parity bits, so the error must have occurred in the message. If the parity is correct, then we can identify the bit with parity information. Firstly, since parity bit 1 is 0, we have to know that either message bit 2 or 4 must be flipped (but not both). Since parity bit 2 is 1, parity bit 3 or 4 must both be flipped, or neither is changed. Finally, parity bit 3 is one, meaning that either message bits 2, 3, and 4 are all flipped, or just one of them is. We know that we can't flip message bits 2, 3, and 4 from parity bit 1, meaning we can only flip one. Since parity bit 2 tells us that flipping message bit 3 or 4 would require flipping both, we deduce that message bit 2 must be flipped  cannot be 1, so message bit 3 must be 1, thus giving us the corrected codeword 1100011. 

To make things easier for ourselves, we can think of the parity as a venn diagram, and use the diagram to help us correct messages more quickly.

<a style="text-align: center">![Parity Venn Diagram](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Hamming%287%2C4%29.svg/300px-Hamming%287%2C4%29.svg.png)</a>

In the above diagram $$d_1 - d_4$$ are our message bits, while $$p_1 - p_3$$ are our parity bits. For any codeword placed in the diagram above, each circle will have even parity. If there has been an error, no circle will have even parity. If it is possible to change a single bit to ensure each circle has even parity, then you know there has only been one error, and which bit it is. Otherwise, you know the transmitted string cannot be corrected. I find the venn diagram much faster than working logically like in the example, however the easiest thing to do is to get a computer to do it for you :).

That's all for now. Hopefully I'll get a chance to write more about error correcting codes and binary linear codes soon.