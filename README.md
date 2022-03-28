# Portfolio
July 2021

https://kristinezheng.github.io/Portfolio/


# Overview

This is my demonstration video:

[demonstration video](https://youtu.be/zjk-ZsDbs1U)

For this design exercise, I used two FSM, one for keeping track of how many claps have occured with 4 states and one for keeping track of the LED screen display with 3 states. By using certain conditions for checking valid claps, the clap state will change and then the LED screen display state will change. 


Specifically, for determining claps, I wired the speaker to my breadboard and read the sound data using analogRead. I removed the offset term of 1.25 V before using a running average filter to reduce the noise and get a signal proportional to the audio signal's envelope. By observing the serial monitor, I determined a threshold for the running average that would count as a clap. 


Next, I use the FSMs for clap state to determine how many claps had occurred. The clap_count gets called in the loop after the audio signal is read in, processed, and averaged. I use 2 timers, one to keep track of the refractory time or the time since the last LED screen change (end of clap sequence) and one to keep track of the time of the last clap. 


This is what happens at every state of the clap_count FSM. 


* At state 0 aka CLAP0 have occured, the screen is still black (from setup). Once the audio output is greater than the threshold, we move to state CLAP1 and the time_of_clap is recorded. 


* At CLAP1, if the time since the last clap is between 0.1 to 1 second and the audio output is greater than threshold, we move to CLAP2 or record 2 claps and the time_of_clap is updated. However, if the time since the last clap is greater than 1 second and no clap had occured, we move back to CLAP0 or 0 claps and reset the refractory_timer. To account for yelling, if the time since the last clap is between 0.15 and 0.5 seconds but the audio output passes the threshold, we go back to CLAP0 since that would mean the sound hasn't stopped for enough time to count as a clap.


* At CLAP2, similar to before, if the time since the last clap is between 0.5 and 1 second and the audio output exceeds the threshold, we move to CLAP3 and update time_of_clap. If the time since the last clap exceeds 1 second and the threshold is not exceeded, we count the overall process as 2 claps and change the LED screen by calling the screen switch FSM and reset the refractory timer. 


* At CLAP 3, we automatically call the screen switch FSM, switch to clap state CLAP0 and update the refractory timer. 


Next, this is what occurs in the screen switch FSM. 


* At state OFF, the initial state, the screen starts out black. If the clap state is CLAP2, I change the led state to RED and fill the screen with TFT_RED. If the clap state is CLAP3, I change the led state to BLUE and fill the screen with TFT_BLUE. At the end, I reset the Clap state to 0 for the next LED screen change. 

* At state RED, the initial state, the screen starts out red. If the clap state is CLAP2, I change the led state to OFF and fill the screen with TFT_BLACK, indicating off. If the clap state is CLAP3, I change the led state to BLUE and fill the screen with TFT_BLUE. At the end, I reset the Clap state to 0 for the next LED screen change. 

* At state BLUE, the initial state, the screen starts out blue. If the clap state is CLAP2, I change the led state to RED and fill the screen with TFT_RED. If the clap state is CLAP3, I change the led state to OFF and fill the screen with TFT_BLACK. At the end, I reset the Clap state to 0 for the next LED screen change. 

## Resources

This is my running average filter that takes an input and averages with the stored values given an order. 

```cpp

float averaging_filter(float input, float* stored_values, int order, int* index) { //running average 

  if (order == 0){
    return input;
  }
  
  float output = input;
  
  for(int i = 0; i<=order; i++){
    output += stored_values[i];
  }
  
  stored_values[*index] = input; 
  
  *index = (*index+1)%order;
  
  return output/(order+1);
}
```

# Summary

You should use your writeup as a way to help us in grading your design exercise. Our job is not to try to figure out what you did. Our job is to grade. Your job is to explain what you did and show it. 


