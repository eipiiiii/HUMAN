"""
MIT BWSI Autonomous RACECAR
MIT License
racecar-neo-prereq-labs

File Name: cone_slalom_sample.py 

Title: Cone slalom sample code

~ added by Erika ~
サンプルコードから青いコーンのコーンスラロームのみを取り出してきたconesから、さらに独自の方法

"""

########################################################################################
# Imports
########################################################################################

import sys
import cv2 as cv
from nptyping import NDArray
import numpy as np
from typing import Any, Tuple, Optional
from enum import Enum, auto


# If this file is nested inside a folder in the labs folder, the relative path should
# be [1, ../../library] instead.
sys.path.insert(1, "../../library")
import racecar_core
import racecar_utils as rc_utils
import math

########################################################################################
# Global variables
########################################################################################

rc = racecar_core.create_racecar()

# Declare any global variables here
# Color of cone
BLUE = ((0, 50, 50), (20, 255, 255))  # The HSV range for the color blue

# Speeds
MAX_ALIGN_SPEED = 0.8
MIN_ALIGN_SPEED = 0.4
PASS_SPEED = 0.5
FIND_SPEED = 0.2
NO_CONES_SPEED = 0.4

# Times
REVERSE_BRAKE_TIME = 0.3
SHORT_PASS_TIME = 1.5
LONG_PASS_TIME = 1.8

# Cone finding parameters
MIN_CONE_AREA = 100
MAX_DISTANCE = 250
REVERSE_DISTANCE = 100
STOP_REVERSE_DISTANCE = 80

FAR_DISTANCE = 120
CLOSE_DISTANCE = 30

# Coefficients of distance estimate function
COEFFICIENT_A = 1.305e05
COEFFICIENT_B = 4.881e01

# LIDAR check angle
FRONT_ANGLE = 10


# mode for cone slalom
class Cone_slalom_mode(Enum):
    align_blue_cone = auto()
    pass_blue_cone = auto()
    reverse_blue_cone = auto()
    find_no_cones = auto()
    return_pass = auto()


# >> Variables
blue_center = None,None
blue_distance,previous_blue_distance,time_counter = 0,0,0,0,0
cur_mode = Cone_slalom_mode.find_no_cones
cone_dist = 0
speed = 0
angle = 0
blue_area = 0


########################################################################################
# Functions
########################################################################################


def find_cones():
    global blue_center,blue_distance,previous_blue_distance,blue_area

    previous_blue_distance = blue_distance

    color_image = rc.camera.get_color_image()
    
    if color_image is None:
        blue_center = None
        blue_distance = 0
        return

    # Search cone
    blue_center, blue_distance ,blue_area= find_cone(color_image, *BLUE)
    

def find_cone(
    color_image: NDArray,
    hsv_lower: Tuple[int, int, int],
    hsv_upper: Tuple[int, int, int],
) ->  Tuple[Optional[Tuple[int, int]], float, float]:

    contours = rc_utils.find_contours(color_image, hsv_lower, hsv_upper)
    largest_contour = rc_utils.get_largest_contour(contours, MIN_CONE_AREA)

    if largest_contour is not None:
        contour_center = rc_utils.get_contour_center(largest_contour)
        distance,area = estimate_distance(largest_contour)
        if distance > MAX_DISTANCE or contour_center is None:
            contour_center = None
            distance = 0
            area = 0
        else:
            rc_utils.draw_contour(color_image, largest_contour, rc_utils.ColorBGR.green.value)
            rc_utils.draw_circle(color_image, contour_center, rc_utils.ColorBGR.green.value)

    else:
        contour_center = None
        distance = 0
        area = 0

    rc.display.show_color_image(color_image)

    return (contour_center, distance,area)


def estimate_distance(contour: NDArray) -> float:
    contour_area = rc_utils.get_contour_area(contour)
    distance = 0

    if contour_area is None:
        return distance

    # function => y = a / x + b
    distance = COEFFICIENT_A / contour_area + COEFFICIENT_B

    if distance < 70:
        distance = 0
    return distance,contour_area

    

def cone_slalom() -> Tuple[float, float]:
    global cur_mode,time_counter

    find_cones()

    scan = rc.lidar.get_samples()
    
    angle: float = 0
    speed: float = 0


    if cur_mode == Cone_slalom_mode.align_blue_cone:
        if blue_center is None or blue_distance == 0 or blue_distance - previous_blue_distance > CLOSE_DISTANCE:
            if 0 < previous_blue_distance < FAR_DISTANCE:
                time_counter = max(SHORT_PASS_TIME, time_counter)
                cur_mode = Cone_slalom_mode.pass_blue_cone
            else:
                cur_mode = Cone_slalom_mode.find_no_cones
        
        else:
            goal_point = rc_utils.remap_range(
                blue_distance, CLOSE_DISTANCE, FAR_DISTANCE, rc.camera.get_width(), rc.camera.get_width() * 3 // 4, True
            )

            angle = rc_utils.remap_range(blue_center[1], goal_point, rc.camera.get_width() // 2, 0, -1)
            angle = rc_utils.clamp(angle, -1, 1)

            speed = rc_utils.remap_range(
                blue_distance, CLOSE_DISTANCE, FAR_DISTANCE, MIN_ALIGN_SPEED, MAX_ALIGN_SPEED, True)


    elif cur_mode == Cone_slalom_mode.pass_blue_cone:
        angle = rc_utils.remap_range(time_counter, 1, 0, 0, 0.5, True)
        speed = PASS_SPEED
        time_counter -= rc.get_delta_time()

        if time_counter <= 0:
            cur_mode == Cone_slalom_mode.return_pass
        

    elif cur_mode == Cone_slalom_mode.return_pass:
        angle = rc_utils.remap_range(time_counter, 1, 0, 0, -0.5, True)
        speed = PASS_SPEED


    elif cur_mode == Cone_slalom_mode.find_no_cones:
        angle = 0
        speed = NO_CONES_SPEED

        if blue_distance > 0 :
            cur_mode = Cone_slalom_mode.align_blue_cone
            
    print("center:" + str(blue_center),"distance:" + str(blue_distance),"area:" + str(blue_area),time_counter)
    print(f"Cone_mode: {cur_mode.name}")

    return speed, angle


# [FUNCTION] The start function is run once every time the start button is pressed
def start():
    global cur_mode

    rc.drive.stop()
    cur_mode = Cone_slalom_mode.find_no_cones


# [FUNCTION] After start() is run, this function is run once every frame (ideally at
# 60 frames per second or slower depending on processing speed) until the back button
# is pressed
def update():
    angle ,speed = 0,0
    if rc.controller.is_down(rc.controller.Button.B):
        speed = -0.5
        angle =0

    if rc.controller.is_down(rc.controller.Button.X):
        speed = 0.5
        angle = -0.5

    if rc.controller.is_down(rc.controller.Button.Y):
        speed = 0.5
        angle = 0.5
    if rc.controller.is_down(rc.controller.Button.A):   
        speed, angle = cone_slalom()
    # cone_slalom()


    rc.drive.set_speed_angle(speed, angle)


# [FUNCTION] update_slow() is similar to update() but is called once per second by
# default. It is especially useful for printing debug messages, since printing a
# message every frame in update is computationally expensive and creates clutter
def update_slow():
    pass  # Remove 'pass and write your source code for the update_slow() function here


########################################################################################
# DO NOT MODIFY: Register start and update and begin execution
########################################################################################

if __name__ == "__main__":
    rc.set_start_update(start, update, update_slow)
    rc.go()