
-----

# SoC Design Adventure: Week 2 Learnings

Welcome to my Week 2 summary\! This week, I took a deep dive into the fascinating world of System-on-Chip (SoC) design. We went beyond just writing code and learned about the complete journey from an abstract idea to a concrete, verifiable digital circuit.

My hands-on lab for this adventure was the awesome **VSDBabySoC** project\!

### The Big Picture: From Blueprint to Reality

The core lesson of this week was understanding the two critical stages of design verification:

1.  **Pre-Synthesis Simulation (Checking the Blueprint):** First, I write our design in a high-level language (Verilog RTL). I then run a simulation to make sure our *logic* is correct. Is our blueprint for the house designed correctly?

2.  **Post-Synthesis Simulation (Checking the Final Product):** Next, a magical tool called a synthesizer (like **Yosys**) takes our blueprint and converts it into a list of real, physical electronic components (standard cells or logic gates). I then run a *second* simulation on this real-world version to prove that it still works exactly like our original blueprint. Does the final house match the blueprint in every way?

Only when both simulations match can we be confident our design is correct\!

------------
The mission was successful because the post-synthesis waveform **perfectly matched** the pre-synthesis waveform. This confirmed that our digital blueprint was successfully translated into a verifiable, physical design\!

### Conclusion

Week 2 was a huge step forward\! I learned that chip design is a process of careful verification at every stage. By understanding both pre-synthesis and post-synthesis simulation, I've built a solid foundation for tackling even more complex hardware designs in the future.
