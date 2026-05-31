---
title: "BubbleID: A Deep Learning Framework for Bubble Interface Dynamics Analysis"
date: 2026-06-01T1:00:00+06:00
draft: false
categories: ["reading"]
tags: ["Deep Learning", "Phase Change", "Heat Transfer"]
math: true
showToc: true
TocOpen: false
---
# **BubbleID: A Deep Learning Framework for Bubble Interface Dynamics Analysis** 

Paper link: *https://pubs.aip.org/aip/jap/article/136/1/014902/3300686/BubbleID-A-deep-learning-framework-for-bubble* 

Replication status: *In progress* 

---

**The Core Novelty: What Is New in This Paper?** 

Most older software looks at boiling water like a photo album—counting bubbles or measuring their size in one frozen picture at a time. But boiling is a fast, messy movie. BubbleID  built to actually *watch* the movie. It follows how bubbles grow and move frame-by-frame.

Here is exactly what this framework brings to the table:

* **Measuring the Bubble's "Skin" (Interface Velocity):** This is the biggest breakthrough of the paper. If a bubble is growing on a hot heater, its center might not move at all, which confuses old software. BubbleID does something brand new: it measures exactly how fast the outer wall (or skin) of the bubble is pushing outward into the water. It achieves this by using a spatial search algorithm called **`cKDTree`** to track hundreds of tiny points on the bubble's surface, calculating the exact speed and direction of the expansion between frames   
* **Not Getting Lost in the Chaos (Advanced Tracking):** Boiling is messy. Bubbles wobble, smash together, and hide behind each other. Old software would lose track of a bubble if it changed shape or got blocked. BubbleID uses a smarter tracker (OC-SORT)\[famous CVPR2023 paper\]  that predicts where a bubble is going. This keeps a strong lock on individual bubbles even when the video gets extremely crowded.  
* **Spotting the Danger Zone (CHF):** By tracking the bubble's "skin," BubbleID detects the Critical Heat Flux (CHF)—the exact moment a boiling system goes from safe to dangerously failing 

***Note on Scope*** 

This paper covers a broad range of topics, and the following discussion focuses on the core physics and methods. Several aspects of the paper are not addressed here in detail, including: tests on copper microchannel and copper foam surfaces; the attached vs. detached bubble classification model (which achieved 95% accuracy); bubble departure rate analysis; and the full machine learning evaluation metrics. These are all covered in the original paper. 

**THE BASICS(REQUIRED TO UNDERSTAND THE PAPER)**

Modern engineering applications: data centers and high-performance electric vehicle batteries. These systems generate staggering amounts of heat in very small physical spaces.

* **The Problem:** If we cannot pull that heat away fast enough, components throttle their performance.

### **Single-Phase Cooling: Reaching Its Limit** 

Historically, we have cooled electronics using **single-phase cooling**. This means blowing cold air over a heat sink or pumping cold liquid (like water or coolant) through a cold plate.

* **The Physics:** The fluid absorbs heat and gets hotter. Relying entirely on the fluid's specific heat capacity (**$c\_p$** ). The formula governing this is sensible heat: **$Q \= mc\_p\\Delta T$**   
* **The Limit:** To remove more heat, we either need a massive temperature difference **(t  )** or we have to pump a huge amount of mass (**m**) very quickly. Single-phase cooling is simply running out of physical capacity to keep up with next-generation processors.**For example**:**Blackwell GB200:** These processors have a massive power draw reaching upwards of **1,200 W to 2,700 W** depending on the configuration, producing a continuous heat flux of **\>50 W/cm²** with thermal hotspots spiking to a staggering **150 W/cm²** 

### **Two-Phase Cooling: The Latent Heat Advantage** 

This is where **two-phase cooling** comes in. Instead of just heating a liquid up, We allow the liquid to *boil* into a gas on the hot surface.

* **The Physics:** When a fluid changes phase from liquid to vapor, it absorbs a monstrous amount of energy without changing its temperature. This is known as the **Latent Heat of Vaporization** ($h\_{fg}$)  .

**This is where pool boiling come into scene:**

**Immersion Cooling** is the method of dunking completely bare computer parts directly into a bath of special, safe liquid to keep them cool.**pool boiling** is one of them

**Pool Boiling** is the natural process where the hot chips make that liquid boil into bubbles, which then float up and carry the heat away.Because the liquid isn't pumped and just sits in a still pool, these two concepts work together to cool servers without needing fans

### **1\. The Holy Grail: The Nucleate Boiling Regime**

As we are talking about pool boiling,we should know about **Boiling Curve (or Nukiyama curve)**This curve plots the heat flux (how much heat you are pulling away) against the temperature of the wall.

                                 **Fig:Nukiyama curve(source:Research Gate)**

The absolute "sweet spot" on this curve is the **Nucleate Boiling Regime**. In this phase, the metal surface isn't entirely covered in vapor. Instead, individual bubbles rapidly form, suck up massive amounts of heat, detach, and carry that heat away into the cooler liquid above. The goal of advanced thermal management is to push hardware as deep into this regime as possible without crossing the limit.

### 

### **2\. Bubble Formation: The Physics of Nucleation** 

Bubbles do not just form randomly in perfectly smooth water. They require a starting point, known as a nucleation site.

If we look at a polished copper heat sink under a microscope, it is actually full of tiny canyons, scratches, and cavities**(in easy word surface roughness)**. Small pockets of vapor get trapped in these microscopic cavities. **As the metal heats up, the trapped vapor expands**.

For a bubble to actually bulge out of the cavity, the pressure inside the vapor ($P\_v$) must overcome the pressure of the surrounding liquid ($P\_l$) plus the surface tension ($\\sigma$) trying to crush it back down. This is governed by the **Young-Laplace equation**:

**$$P\_v \- P\_l \= \\frac{2\\sigma}{r}$$**

***(Where $r$ is the radius of the bubble).***

### 

### ***Once the bubble begins to grow, three competing physical processes govern its behaviour***

### **The Inflation (Pressure)*:***the trapped vapor inside the microscopic scratch gets heated and wants to [expand.**we**](http://expand.we) **know that from Pv=nRT**. The internal vapor pressure ($P\_v$) pushes outward against the surrounding liquid. As long as the pressure inside is strong enough to overcome the surface tension trying to crush it and liquid pressure which is equal to **Pl= h\*row\*g(h\*p\*g)**  the bubble inflates.

### 

### **The Lift (Buoyancy):**As the bubble inflates, it takes up more space (its volume increases). Because vapor is much lighter (less dense) than the liquid surrounding it, it displaces the heavy liquid.**This triggers Archimedes' Principle**: the surrounding liquid pushes back, creating an upward Buoyant Force. The bigger the bubble gets, the stronger this upward lift becomes.

### 

### 

### ***Detachment(Tug-of-War) :***While the bubble is growing, there is a literal tug-of-war happening:

* **Pulling DOWN**: Surface tension acts like a sticky glue, anchoring the base of the bubble to the cavity on the metal surface.  
* **Pulling UP**: Buoyancy is yanking the bubble upward, and this upward pull gets stronger every millisecond as the bubble's volume grows.

Eventually, the bubble grows so large that the upward Buoyant Force becomes stronger than the downward Surface Tension. Once that threshold is crossed, the bubble violently snaps off the cavity and floats up to the surface. A tiny bit of vapor is left behind in the scratch, acting as the "seed" for the next bubble to start growing, and the cycle repeats\!

### ***The Lifecycle of a Single Bubble*** 

**Stage 1 — Birth (Nucleation)**: Trapped vapor in a microscopic surface cavity builds sufficient pressure to emerge onto the heater surface *.*

**Stage 2 — Growth (Still Attached)**: The bubble sits on the heater surface as a dome. The liquid immediately surrounding the bubble is superheated and rapidly evaporates into the vapor space, causing the bubble to inflate 

**Stage 3 — Detachment**: The bubble continues to grow until the buoyant force exceeds surface tension adhesion, at which point it snaps free and rises through the liquid. 

### 

### 

### 

### 

### **3\. Growth and the Interface Velocity: Where the Novel Metric Comes In** 

Once the bubble bulges out**(bulging doesnt mean detached)**, it enters the growth phase. Here is where the paper's core novelty—Interface Velocity—comes into play.

The liquid immediately touching the bubble's surface (the interface or "skin") is superheated. As this liquid evaporates into the bubble, the bubble expands. The rate at which this skin **expands outward physically dictates how fast the system is absorbing latent heat**.

**Older AI just tracked the center of the bubble, which tells us almost nothing about the heat transfer rate. By tracking the expanding velocity of the interface itself, the BubbleID framework is essentially measuring the real-time thermal absorption of the bubble.**

**Few questions one may ask:**

### **1\. Why is the liquid touching the bubble superheated?**

When you have a heated metal plate (like a server heat sink), the metal is much hotter than the fluid's boiling point.

The fluid directly touching that metal plate absorbs heat incredibly fast. It happens so quickly that a microscopic, ultra-thin layer of liquid right against the metal shoots past the boiling point. This is called the Thermal Boundary Layer. Because the bubble is sitting on the metal, the bottom half of the bubble is surrounded by this extra-hot, superheated layer.

### **2\. If it is superheated, why isn't it vapor?**

This is the big secret of phase change: Liquid molecules cannot just turn into gas anywhere they want. They need a "doorway."

In a liquid, the molecules are held tightly together by intermolecular bonds. To turn into a gas, they need to violently expand. But if a single molecule in the middle of the superheated liquid tries to turn into gas, the pressure of all the surrounding water and the crushing force of surface tension will instantly squeeze it back into a liquid state.

To overcome that crushing pressure and create a brand-new bubble out of nowhere (called *homogeneous nucleation*), the liquid would have to get absurdly hot—way hotter than standard superheating.

So, the liquid is trapped. It holds onto that extra heat energy, wanting to evaporate, but it has no physical space to expand into.

### **The Bubble is the "Escape Hatch"**

**This is why the bubble's skin (the interface) is so important\!**

When that trapped, superheated liquid bumps into the wall of an existing bubble, it suddenly finds a low-resistance "doorway." There is no longer a crushing wall of water in its way; there is a nice, spacious vapor pocket.

The superheated liquid instantly uses its stored-up energy to flash into vapor, crossing the interface and entering the bubble.

***To tie it all back to this  paper*****:** The superheated liquid is the fuel. The bubble's skin is the engine. As the superheated liquid flashes across that boundary, it pushes the skin outward. That rapid expansion is the Interface Velocity the AI is tracking\!

### 

### 

### 

### **4\. Death: The Departure Point (A Game of Tug-of-War)**

A bubble cannot stay on the heater forever. As it grows, a fierce mechanical tug-of-war begins. As we already talked about previously

act millisecond that Bouyancy force \> surface tension, the bubble snaps off the heater and floats away. This is the Departure Event. **The AI tracks how many times this happens per second to calculate the Departure Rate. A high departure rate means  system is pumping heat away efficiently. *(although later i didnt talk about it)***

### **5\. Chaos: Coalescence**

In a high-powered system, nucleation sites are packed closely together. As bubbles grow, they crash into each other and merge—a process called Coalescence.

When two bubbles merge, their combined surface area drops, which temporarily lowers their heat transfer efficiency, but their combined volume increases, causing a massive spike in buoyancy that yanks them off the heater faster. This chaotic merging is exactly why standard AI tracking software fails, and why this paper needed the advanced **OC-SORT algorithm** to keep the data clean.

### **Incoming Danger in our happy life**

As we increase the heat flux ($q''$), the microscopic cavities start pumping out bubbles faster and faster. At a certain point, the bubbles are forming so rapidly that they don't have time to float away before the next one forms.(as we know they have to increase in size to increase buoyancy force.but it needs time.

The bubble begins violently crashing into each other. This causes massive **Coalescence** (merging) right on the surface of the heater. Instead of distinct, round bubbles, we get massive, chaotic "mushroom clouds" of vapor. The liquid from above is trying to flow down to cool the heater, but the massive vapor columns are blocking the way. 

### **2\. The Limit: Critical Heat Flux (CHF)**

As the heat keeps rising, we hit an absolute physical ceiling known as the **Critical Heat Flux (CHF)** or the *boiling crisis.* we can see that in boiling curve.after that heat flux decrease

CHF ($q''\_{max}$) is the maximum amount of thermal energy the boiling liquid can possibly absorb before the system completely changes its physical state. Up until this exact microsecond, the boiling has been helping to cool the system. The moment we cross CHF, boiling becomes our worst enemy.

### **3\. The Blanket: Film Boiling**

When the CHF limit is crossed, the layer of vapor completely merge together. They form a continuous, unbroken layer of gas across the entire surface of the heater. This is called the **Vapor Blanket** (or the Film Boiling regime).

Why is this so dangerous? Because of **thermal conductivity** ($k$).

* Liquid water is a decent conductor of heat.  
* Vapor (gas) is a terrible conductor of heat; it acts as a highly effective **insulator**.

By boiling the liquid too violently, we have accidentally wrapped our incredibly hot electronics in a microscopic thermal blanket. The liquid water is still floating on top of this blanket **(a phenomenon known as the Leidenfrost effect)**, but it can no longer touch the actual metal surface to cool it down.

### **4\. The Consequence: Burnout**

Because the heat flux ($q''$) from the electronic chip is still pouring into the metal, but the vapor blanket is blocking that heat from escaping into the liquid, the heat has nowhere to go.

The temperature of the metal wall ($T\_{wall}$) spikes instantaneously and uncontrollably. Within milliseconds, the temperature can shoot up hundreds or even thousands of degrees. This sudden, violent temperature spike is called **Burnout**. In the real world, this means the silicon chip melts

**How This Paper Works Step By Step**

---

**Step 1 — Instance Segmentation (Mask R-CNN)** 

To measure heat transfer accurately, we need the exact physical area of the bubble, not just a general location.

BubbleID uses a deep learning model called **Mask R-CNN** to perform "Instance Segmentation." **They used a state-of-the-art,Detectron2 open-source computer vision library developed by Meta .**The AI drawing a perfect, tight mask over the exact, irregular shape of the bubble.

### 

### **Step 2 — Multi-Object Tracking (OC-SORT)** 

As boiling intensifies, bubbles rapidly merge (coalescence) and hide behind one another. To gather useful time-lapse data, the AI must continuously keep track of which bubble is which.

BubbleID uses an algorithm called **OC-SORT** (Observation-Centric Simple Online and Real-Time Tracking). This algorithm actively calculates each bubble's momentum, speed, and direction. If a bubble temporarily hides behind another or merges into a larger vapor cloud, OC-SORT uses these physics calculations to predict where the bubble is going, maintaining its distinct identity and data stream through the chaos.

### **Step 3 — Interface Velocity Calculation (cKDTree)** 

Once the AI has the perfect mask (from Step 1\) and knows it is tracking the exact same bubble across time (from Step 2), it measures the **Interface Velocity**.

It achieves this using a highly efficient spatial algorithm called **cKDTree**. The algorithm takes hundreds of tiny coordinate points along the bubble's skin in Frame 1\. It then looks at Frame 2 (a fraction of a second later) and rapidly finds the nearest corresponding points on the newly expanded skin. By measuring the distance between these points, it calculates exactly how fast the boundary is stretching outward.

### **The Mathematics Behind the Framework**   **1\. Heat Flux Measurement: Fourier's Law of Heat Conduction** 

Before looking at the bubbles, the researchers needed to know exactly how much heat was *actually* leaving the copper heater. To measure this "ground truth" heat flux, they drilled holes into the side of the solid copper block and inserted four temperature sensors (thermocouples) spaced exactly 0.1 inches apart.

They calculated the heat flux ($q''$) passing through the copper and into the water using standard **1D steady-state heat conduction**:

$$q'' \= \-k \\frac{dT}{dx}$$

* **$q''$**: The Heat Flux (measured in $W/cm^2$). This is what they are ultimately trying to maximize.  
* **$k$**: The thermal conductivity of the copper block.  
* **$\\frac{dT}{dx}$**: The temperature gradient. Because they have four sensors spaced apart ($dx$), they can measure how much the temperature drops ($dT$) as the heat travels up the block toward the water.  
  **(WE ALL DID THIS PRACTICAL IN OUR MECHANICAL ENGINEERING COURSE)**  
  **FINALLY SOMETHING WE LEARNT CAN APPLY IN OUR REAL LIFE.**

### 

### 

### **2\. Interface Velocity** 

This is the math used to calculate how fast the bubble's "skin" is stretching into the fluid. It is a fundamental physics calculation of velocity (distance over time).

$$\\vec{v}\_{interface} \= \\frac{\\Delta \\vec{d}}{\\Delta t}$$

* **$\\Delta \\vec{d}$**: The physical distance a tiny point on the bubble's skin moved between two video frames. (The software counts the pixels, and then multiplies it by a scale factor to convert it into actual physical centimeters).  
* **$\\Delta t$**: The time elapsed between the two frames. (Since the camera shoots at 3,000 frames per second, $\\Delta t$ is incredibly small, allowing them to capture microsecond-level physics).

### **3\. Equivalent Diameter** 

Because boiling bubbles wobble and deform (they look like squished ovals, not perfect spheres), We cannot just measure them with a ruler. To find the standard physical size of a wobbly bubble, they use the **Equivalent Diameter** ($d\_{eq}$).

This math takes the total 2D area of the wobbly bubble and calculates the diameter of a *perfect circle* that has that exact same area ($Area \= \\frac{\\pi d^2}{4}$).

$$d\_{eq} \= \\sqrt{\\frac{4N}{\\pi \\alpha^2}}$$

* **$N$**: The total physical area of the bubble (calculated by counting the number of pixels inside the bubble).  
* **$\\alpha$**: A scale factor representing the physical resolution of the camera (how many pixels fit into 1 centimeter).  
* **$d\_{eq}$**: The final physical diameter of the bubble in centimeters.

### **4\. Vapor Fraction** 

In thermodynamics, vapor is a thermal insulator. The more vapor on the surface, the harder it is to transfer heat. Therefore, engineers need to know the **Vapor Fraction** ($VF$), which is the ratio of gas to total space.

$$VF \= \\frac{A\_{vapor}}{A\_{total}}$$

* **$A\_{vapor}$**: The total surface area currently covered by boiling bubbles (insulator).  
* **$A\_{total}$**: The total surface area of the image (or heater).  
* **The Physics Meaning:** If the Vapor Fraction is low, the liquid is touching the heater, and heat transfer is excellent. As the Vapor Fraction approaches 1.0 (100%), the system is hitting Critical Heat Flux (CHF) and a dangerous vapor blanket is forming

**5\. Heat Flux Prediction: Hybrid Regression Model**  

Once the visual tracking is complete, the BubbleID framework uses a final **Machine Learning HYbrid Regression Model** to bridge the gap between video footage and physics. Instead of relying on traditional thermodynamic equations to calculate thermal energy, this predictive algorithm acts as a highly advanced pattern-recognizer. It takes four specific metrics extracted from the video frames as its inputs: **Vapor Fraction, Bubble Count, Minimum Bubble Size, and Maximum Bubble Size**. By analyzing thousands of frames during training, the AI learns the complex correlation between the physical geometry of these bubbles and the actual thermal energy leaving the heated surface. Ultimately, this allows the framework to take a live video feed of boiling liquid and accurately output the real-time **Heat Flux**, proving that tracking the visual behavior of bubbles is a highly reliable way to measure thermodynamic performance. 

**Extra:**  
**Hybrid Regression Model:**

Standard machine learning models usually only take one type of input—like a spreadsheet of numbers. But boiling is so complex that just having four numbers isn't enough to capture the full physical reality.

To get that incredibly accurate 3.99% error margin, the AI actually has two "eyes" looking at two different streams of data at the exact same time:

Input 1: The Extracted Numbers (The Math) This is the data we talked about earlier. The framework feeds the model the exact calculated numbers:

* Vapor Fraction  
* Bubble Count  
* Minimum Bubble Size  
* Maximum Bubble Size  
* 

**Input 2: The Raw Image (The Spatial Context) At the exact same time, it feeds the raw,** pixel-by-pixel image from the camera straight into the model. Why? Because the *arrangement* matters. If we have 40 bubbles, are they spread out evenly, or are they all clustered violently in one corner? The four numbers can't tell us that, but the raw image can.

How it comes together: The hybrid model combines both streams. It uses the raw image to understand the spatial arrangement and chaos of the boiling, and it uses the four exact numbers to ground those visuals in hard geometry. By looking at the picture *and* the math simultaneously, it can predict the heat flux with near-perfect accuracy.

**Key Finding: Detecting the Approach to CHF** 

Predicting the exact heat flux is impressive, but the ultimate goal of BubbleID was to answer one critical engineering question: *How do we know when the system is about to break?* The researchers found the answer by looking at their brand-new metric: the **Interface Velocity**. Just before failure, the bubbles essentially panic. The AI discovered that right before the system hits the Critical Heat Flux limit (pre-CHF), the bubble's "skin" expands violently—reaching peak speeds nearly **three times faster than the bubbles that form *after* the vapor blanket has already choked the system (post-CHF)**. By continuously monitoring for this massive, **3x spike in interface velocity**, BubbleID acts as a microscopic alarm system . It doesn't need to wait for a physical temperature sensor to overheat; the moment the AI sees that massive spike in interface velocity, it knows the "danger point" is imminent, giving engineers the exact signal they need to throttle the power before catastrophic failure occurs. 

