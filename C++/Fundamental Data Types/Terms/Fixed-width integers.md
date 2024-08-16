- In the early days of C, when computers were slow and performance was of the utmost concern. C opted to intentionally leave the size of an integer open so that the compiler implementers could pick a size for int that performs best on the target computer architecture.
- To address the above issues, C99 defined a set of **fixed-width integers** (in the stdint.h header) that are guaranteed to be the same size on any architecture.

|Name|Type|Range|Notes|
|---|---|---|---|
|std::int8_t|1 byte signed|-128 to 127|Treated like a signed char on many systems.|
|std::uint8_t|1 byte unsigned|0 to 255|Treated like an unsigned char on many systems.|
|std::int16_t|2 byte signed|-32,768 to 32,767||
|std::uint16_t|2 byte unsigned|0 to 65,535||
|std::int32_t|4 byte signed|-2,147,483,648 to 2,147,483,647||
|std::uint32_t|4 byte unsigned|0 to 4,294,967,295||
|std::int64_t|8 byte signed|-9,223,372,036,854,775,808 to 9,223,372,036,854,775,807||
|std::uint64_t|8 byte unsigned|0 to 18,446,744,073,709,551,615||