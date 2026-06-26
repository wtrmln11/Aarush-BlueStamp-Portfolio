# Facial Recognition System
This project uses a Raspberry Pi to build a real-time facial recognition system capable of identifying people and sending email notifications when a face is detected. Using OpenCV and machine learning libraries, the Pi is trained on a custom image dataset to recognize specific individuals through a connected camera. 
You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| AarushH | Evergreen Valley Highschool | Computer Engineering | Incoming Senior |

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
<!--  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 
-->
# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/wKA9XhxVHsE?si=93JbfTmWjfTUcsn5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your first milestone, describe what your project is and how you plan to build it. You can include:

# Starter Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/wKA9XhxVHsE?si=93JbfTmWjfTUcsn5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


My project is the retro arcade project. There is a screen of some sort and the way it powers on is from the battery that I had to solder to the metal connectors on the back. I also had to solder the charging port on the side. We also soldered the control buttons, such as movement and the reset. Then we built the acrylic case and isntalled screws which I had to improvise since I was missing some.

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++
void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("Hello World!");
}

void loop() {
  // put your main code here, to run repeatedly:

}
```

# Bill of Materials

Here's where you'll list the parts in your project.

| **Part** | **Note** | **Price** | **Link** |

|:--:|:--:|:--:|:--:|

| 10.1" Security Monitor | External display for Raspberry Pi | Used to display raspberry pi on to screen | <a href="https://www.amazon.com/Haiway-Security-Surveillance-Controller-Resolution/dp/B07WKG9J35?th=1"> Link </a> |

| Raspberry Pi Starter Kit | Includes Raspberry Pi, microSD card, power supply, and case | Includes neccesary components such as the raspberry pi, micro sd card with Raspberry OS, power, and case to hold the pi| <a href="https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9/ref=sr_1_1?dib=eyJ2IjoiMSJ9.4tX4qJd4-AxDDD69js_G-klIDZ_9KAfVg_zuk3y_OXMYIy2624zr8ofr_O6RNfaXyIeh-VUizY3kxGSUcMT8NgbafM_JrlJFfHo9OB4eAVE812W94Jh_RVxkFR2mGtyS0Kq1Wm3JLtfQisU8xWPqH0jDKX1A4X7ixHgTtXGqtZmcTSyp3PojN0l-i2zo0osHG2l7xi5wbJ-c10xT_FpXw70iYD6DARRYdCWdUNsA0Qw.pOQ4Vh38_1tedL4XXuN18IxgC72no0BhC0Bw9bZmcpI&dib_tag=se&keywords=raspberry%2Bpi%2Bstarter%2Bkit&qid=1782329275&sr=8-1&th=1"> Link </a> |

| Raspberry Pi Camera Module | Camera used for facial recognition | Used to take the photos| <a href="https://www.amazon.com/Raspberry-Pi-Camera-Module/dp/B0BRY6MVXL/ref=sr_1_1_mod_primary_new?crid=16XMM643GLZ80&dib=eyJ2IjoiMSJ9.6WrYADCMjOY9gW5mqaDlpKL5UUOrsJ5iu3sNf2dgqPtrSbPVPaqanggClUNg6fMqUZW2Kqy7MxAtrAXv9Oe3OkGzbedyOkyc_K1YT9oLAbV5mfkMNl4L-QTwptbiEB8N1u7Wog2sQgl43x0tXJ9E5z0VBC-RRQeSBBWrWEj6P5tFxen0T0c0d9boJpj6-kB9FlK4ki4Py0nQ-R_wwNQn_qzxid5wNC9FYm3sO10ijnc.TR91zc93GVHjv-oKznWDZx1v3GUbsJP28EIFZugUOJk&dib_tag=se&keywords=raspberry%2Bpi%2B4%2Bplus%2Bcamera%2Bmodule&qid=1782329418&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=Raspberry%2BPi%2BCamera%2BModule%2Bfor%2Bpi%2B4%2Caps%2C504&sr=8-1&th=1"> Link </a> |

| Keyboard and Mouse | Input devices for setup | To navigate the interface of Raspberry OS | <a href="https://www.amazon.com/Logitech-Keyboard-Windows-Optical-Full-Size/dp/B003NREDC8/ref=sr_1_3?crid=24J0NUL8LPAFQ&dib=eyJ2IjoiMSJ9.Y-nH--Ry5RWB99O7EtO5uBhuM-vynJwbsvF0tTrC6da1iPKmxkxWURJPPMaa9zBJcGlwCSZJnJorFUq_Sc4koMwH3GGO8nJpBbQ-eVfgvDRaM2-CRCVIhpW6Q0NLJo-g7xIM4R1GwdbZ3T7AIBOOH1UV1GqL12WXMFYvy6NUpMxJT7RZmz6ycMnlZGZt1i7WE529mSQrNr9kcUk0BXsP5Unb_3sinsskOOp0mCCohz8.0Hhw5x3S1jbTdfdnG1Phscxw5iI2f5S8m0pBw8uU_iE&dib_tag=se&keywords=Keyboard+and+Mouse+wired&qid=1782329452&sprefix=keyboard+and+mouse+w%2Caps%2C546&sr=8-3"> Link </a> |
# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
