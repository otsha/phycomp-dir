---
title: "Week 2"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Alt+Ctrl (Week 2)

Deliverables for the 2nd week's assignments.

## Assignment 1: Existing Alt+Ctrl Interface: VinylOS

VinylOS is a combination of a visual interface and a game controller. Content (likely a game) is projected onto the vinyl. Interacting with the vinyl (rotating/scratching) controls the content.

I was specifically looking for music-themed interactions / games because novel ways of making music are always fascinating to me - even if I don't end up using them in my own work. But what initially caught my attention in this case was the name of the interface - how do you build an "operating system" around vinyls? Looking at the demo video, there are quite a few games / digital toys that can be built around the system!

- [Demo on YouTube](https://www.youtube.com/watch?v=DP7HzmlX1kE)
- [Webpage](https://vinylos.io/)
- [Shakethatbutton showcase](https://shakethatbutton.com/vinylos/)

---

## Assignment 2: Alt+Ctrl Concept: Catbell

> A game where you guide a cat down from a tree, branch by branch, using sound.

## The Premise

- A cat is stuck in a tree :( 
- But! You can use the sound of a tiny bell to make it come down, branch by branch
- Crows land on branches of the tree from time to time
    - If there are crows on a branch, the cat won’t jump on it
- To shoo away the crows, you have to yell (or make loud noises)
    - Yelling also scares the cat back up by one branch :/
- If there are multiple crows on a branch, you have to yell louder to scare them away
    - Yelling louder scares the cat up by multiple branches
- Good luck I guess? Should just call the fire brigade...

## The Controller

- A tiny bell (or a jinglebell)
- Arduino nano connect RP2040 w/ microphone

## Conceptualizing the Implementation

The Arduino [PDM library](https://docs.arduino.cc/learn/built-in-libraries/pdm) reads microphone input into a buffer in bytes, where every sample (a pair of consequent bytes) seems to contain a byte with a negative and a positive value. Below are absolute byte values that I determined to be acceptable thresholds for the different levels of input.

- Bell threshold: `Above 300, Below 700`
- Yell threshold: `2000`
- Louder yell threshold: `5000`
- Loudest yell threshold: `10000`

The RP2040 microphone seems to be a bit picky about the frequencies it detects. Fine-tuning (and exploration of different bells) should be done.

### Actual Code for Detecting Events

```arduino
#include <PDM.h>

short sampleBuffer[256];
long previousInput;

void setup() {
  Serial.begin(9600);
  PDM.setGain(40);
  PDM.begin(1, 16000); // 16kHz, mono
  previousInput = millis();
}

void loop() {
  int iAvg = readBufferAverage();
  if (millis() - previousInput > 1000) {
    if (iAvg > 300) { // > 300 to filter out unwanted noise
      handleInput(iAvg);
      previousInput = millis();
    }
  }
}

/**
* Read the PDM buffer and return an average value.
* Most sounds take way longer to trail off than you'd expect.
* By reading an average we get a sort-of reliable estimate of
* the volume of sounds that happened in a given timeframe.
*/
int readBufferAverage() {
  int nOfBytes = PDM.available();
  int nOfSamples = nOfBytes / 2;
  PDM.read(sampleBuffer, nOfBytes);

  int sampleSum = 0;
  for (int i = 0; i < nOfSamples; i++) {
    sampleSum += abs(sampleBuffer[i]);
  }

  return sampleSum / max(1, nOfSamples * 2);
}

void handleInput(int inputAverage) {
  if (inputAverage > 10000) {
    Serial.println("loud yell");
    // shoo(3); or what have you (see game logic pseudocode)
  } else if (inputAverage > 5000) {
    Serial.println("yell");
    // shoo(2);
  } else if (inputAverage > 2000) {
    Serial.println("yelp");
    // shoo(1);
  } else if (inputAverage < 700) {
    Serial.println("tingle");
    // attract();
  } else {
    Serial.println("no reaction...");
  }
}
```

### Pseudocode for Game Logic

```python

# Attempt to jump down on the next branch.
# - If there are crows on the next branch, the cat won't jump.
def attract():
    if n_of_crows_on_next_branch == 0:
        jump_down()

# Move the cat down by one branch.
# If there are no more branches below the cat, the ground has been reached.
# ???
# Victory
def jump_down():
    if current_branch > 0:
        current_branch -= 1
    else
        win_game()

# Shoo away crows.
# - Number of crows equal to yell_strength removed from each branch
# - Move the cat's position upwards by one branch
# @param yell_strength: An integer from 1 to 3
def shoo(yell_strength):
    for branch in tree:
        remove_crows(branch, yell_strength)

    jump_up(yell_strength)

# Jump up n branches.
# - If the jump would reach the top of the tree, set current_branch
# to be the highest branch
# @param n_of_branches: The number of branches to jump up
def jump_up(n_of_branches):
    if current_branch + n_of_branches < total_n_of_branches:
        current_branch += n_of_branches
    else
        current_branch = total_n_of_branches

```
