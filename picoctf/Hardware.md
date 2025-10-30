# IQ Test
This was a simple Logic Gates test that we had to solve based on the number we got. 

## Solution 
To solve this challenge, I followed the steps as listed below: 
- I first read the description of the challenge to understand what I had to do. After understanding, I converted the number given to us in decimal to binary and then added it as the input to the logic gates.
- I did it wrong the first time as I assumed that we had to take the rightmost bit as x0, but on doing it the second time around, I got the correct answer.
- The process is shown in the image below:
![Logic Gates Solved](/images/hardware/iqtest.png)

## Flag

`nite{100010011000}`

## References
- https://drive.google.com/drive/folders/1DG7TRZDwHl-X6s4s8ApypMOOYGNNsUJ1?usp=drive_link
---------------------------------------------------------------------------------------------------------------------------------------
# I Like Logic 
To solve this challenge, we had to analyze a file with `.sal` extension and use Logic 2 Software to get the result. 

## Solution 
To solve this challenge, I followed the steps as followed below: 
- I opened the link present in the Briefing, and I found a `challenge.sal` file. I downloaded it and then googled what is a `.sal` extension. Turns out it stands for Saleae Logic 2 capture file.
- So I downloaded the Logic 2 Software and uploaded the file in this. Using the analyzers, I went about manually checking each analyser filter. Lucking I found the right filter on the second try itself, which was the `Async Serial` Analyzer.
- Reading the terminal output, I found a story with some random data of the form `FCSC{...}`. Confirming with mentor, I got this as the correct flag.
![screenshot of Logic 2 Software](/images/hardware/Screenshot-2025-10-30-16-48-49.png))

## Flag
`FCSC{b1dee4eeadf6c4e60aeb142b0b486344e64b12b40d1046de95c89ba5e23a9925}` 

## References
- https://drive.google.com/drive/folders/1ZEhUzJgVHVTMIlBWrPIHCzH4TnipQPNX
- Google Search on `.sal`
------------------------------------------------------------------------------------------------------------------------------------
