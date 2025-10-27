# IQ Test
---------------------------------------------------------------------------------------------------------------------------------------
# I Like Logic 
To solve this challenge, we had to analyze a file with `.sal` extension and use Logic 2 Software to get the result. 

## Solution 
To solve this challenge, I followed the steps as followed below: 
- I opened the link present in the Briefing, and I found a `challenge.sal` file. I downloaded it and then googled what is a `.sal` extension. Turns out it stands for Saleae Logic 2 capture file.
- So I downloaded the Logic 2 Software and uploaded the file in this. Using the analyzers, I went about manually checking each analyser filter. Lucking I found the right filter on the second try itself, which was the `Async Serial` Analyzer.
- Reading the terminal output, I found a story with some random data of the form `FCSC{...}`. Confirming with mentor, I got this as the correct flag.
![screenshot of Logic 2 Software]()

## Flag
`FCSC{b1dee4eeadf6c4e60aeb142b0b486344e64b12b40d1046de95c89ba5e23a9925}` 

## References
- https://drive.google.com/drive/folders/1ZEhUzJgVHVTMIlBWrPIHCzH4TnipQPNX
- Google Search on `.sal`
------------------------------------------------------------------------------------------------------------------------------------
