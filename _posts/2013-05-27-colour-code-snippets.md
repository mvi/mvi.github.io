# Colour Code Snippets


I’ve previously produced a number of mod tools for Dune 2000 that convert colours from various formats to 24 bit and back. Although most of the source code is already online, I thought I’d share the colour conversion C# source code in one place.

## Convert 15 bit high colour to 24 bit true colour

This converts 15 bit (with 1 bit padding) colour to 24 bit, by first extracting the red, green and blue channels. In 15 bit colour, each channel has 5 bits meaning 25 = 32 permutations, but we want 8 bit colour for each channel which is 28 = 256 permutations, so we divide by the highest 5 bit value and multiply by the highest 8 bit value to convert the range from 0-31 to 0-255.

{% highlight csharp %}
{% raw %}
private Color ConvertColor(UInt16 colour)
{
    // bits = XRRRRRGGGGGBBBBB
    byte red = (byte)((colour >> 10) & 0x1f);
    byte green = (byte)((colour >> 5) & 0x1f);
    byte blue = (byte)(colour & 0x1f);
 
    red = Convert.ToByte(255.0f * (float)red / 31);
    green = Convert.ToByte(255.0f * (float)green / 31);
    blue = Convert.ToByte(255.0f * (float)blue / 31);
 
    return Color.FromArgb(red, green, blue);
}
{% endraw %}
{% endhighlight %}

## Convert 24 bit true colour to 15 bit high colour

Similarly to convert to 15 bit, we convert each channel from 8 bit to 5 bit. Then we bit shift the red and green values and combine the three channels using a bitwise OR, so that we can fit all three 5 bit channels in the format XRRRRRGGGGGBBBBB.

{% highlight csharp %}
{% raw %}
private UInt16 ConvertColor(Color colour)
{
    int r = (int)((colour.R / 255.0f) * 31) << 10;
    int g = (int)((colour.G / 255.0f) * 31) << 5;
    int b = (int)((colour.B / 255.0f) * 31);
 
    int combined = r | g | b;
    return Convert.ToUInt16(combined);
}
{% endraw %}
{% endhighlight %}

## Convert 24 bit true colour to existing palette

This algorithm calculates the entry in an existing palette which is closest to the target colour. The algorithm treats colours as points in 3 dimensional space – instead of x,y,z we use r,g,b – and uses Euclidean Distance (the distance in a straight line between two points) to calculate the closest colour. For efficiency, all distances are kept at their square values, since it is not necessary to use the relatively expensive square root operation in distance comparisons. The assumption here is that the palette is 24 bit, however if you have a 15 bit palette you could use the conversion technique described above to convert it to a 24 bit palette.

{% highlight csharp %}
{% raw %}
Color[] palette = new Color[256];
 
int GetNearestColorIndex(Color targetColor)
{
    int index = 0;
    // Set the initial color distance to the maximum
    double currentDistance = (255 * 255) * 3;
    Color paletteColor;
    for (int i = 0; i < palette.Length; i++)
    {
        paletteColor = palette[i];
        int redDistance = paletteColor.R - targetColor.R;
        int greenDistance = paletteColor.G - targetColor.G;
        int blueDistance = paletteColor.B - targetColor.B;
 
        // Calculate the square distance from the target colour to this palette entry
        int distance = (redDistance * redDistance) +
                            (greenDistance * greenDistance) +
                            (blueDistance * blueDistance);
 
        if (distance < currentDistance)
        {
            index = i;
            currentDistance = distance;
        }
    }
    return index;
}

Color GetNearestColor(Color targetColor)
{
    return palette[GetNearestColorIndex(targetColor)];
}
{% endraw %}
{% endhighlight %}
