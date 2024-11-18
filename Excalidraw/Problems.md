```ad-question
Có một biến với kiểu mới trong vision, làm cách nào để có thể giúp truyền biến đó giữa DSC -> Datahandler -> Vision?
![[Pasted image 20241016085829.png|center]]
```

```ad-Answer
**1 - Flow (Old)**
![[image-2024-8-23_15-9-16.png|center]]

**2 - Managed Map**
![[Pasted image 20241016090919.png|center]]

**3 - Hàm Storer và Loader**
![[Pasted image 20241016091506.png|500|center]]
![[Pasted image 20241016091525.png|500|center]]

**4 - Deserialize tại C++ Side**
- Check DcsSerializer.h

**5 - Note**
- Why use NOT operator here?
![[Pasted image 20241016092422.png|center]]
```
