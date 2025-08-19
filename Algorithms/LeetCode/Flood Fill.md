---
tags:
  - algorithms
  - leetcode
---

```csharp
int[][] FloodFill(int[][] image, int sr, int sc, int color)
{
    var originalColor = image[sr][sc];
    if (originalColor == color) return image;

    Fill(image, sr, sc, originalColor, color);

    return image;
}

void Fill(int[][] image, int r, int c, int color, int newColor)
{
    if (r >= image.Length || c >= image[0].Length || r < 0 || c < 0) return;

    if (image[r][c] != color) return;

    image[r][c] = newColor;

    Fill(image, r + 1, c, color, newColor);
    Fill(image, r - 1, c, color, newColor);
    Fill(image, r, c + 1, color, newColor);
    Fill(image, r, c - 1, color, newColor);
}
```