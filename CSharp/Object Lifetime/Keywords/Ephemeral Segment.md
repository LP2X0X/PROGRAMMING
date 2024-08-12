- Generations 0 and 1 are short-lived and are known as [[Ephemeral Segment|ephemeral generations]]. These generations are allocated in a memory segment known as the ephemeral segment.
- As garbage collection occurs, new segments acquired by the garbage collection become new ephemeral segments, and the segment containing objects surviving past generation 1 becomes the new generation 2 segment.

![[Pasted image 20240605104747.png|center]]