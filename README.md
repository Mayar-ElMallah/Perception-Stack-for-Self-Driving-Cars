# Perception-Stack-for-Self-Driving-Cars
In this project we are going to create a simple perception stack for self-driving cars (SDCs.) Although a typical perception stack for a self-driving car may contain different data sources from different sensors (ex.: cameras, lidar, radar, etc...), we’re only going to be focusing on video streams from cameras for simplicity. We’re mainly going to be analyzing the road ahead, detecting the lane lines, detecting other cars/agents on the road, and estimating some useful information that may help other SDCs stacks. The project is split into two phases. We’ll be going into each of them in the following parts.

## Phase 1 - Lane Line detection:
In any driving scenario, lane lines are an essential component of indicating traffic flow and where a vehicle should drive. It's also a good starting point when developing a self-driving car!The lane detection system was written in Python using the OpenCV library. Here's the current image processing pipeline:
1) Distortion Correction
2) Perspective Warp
3) Sobel Filtering
4) Histogram Peak Detection
5) Sliding Window Search
6) Curve Fitting
7) Overlay Detected Lane
8) Apply to Video

## Distortion Correction
![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)
