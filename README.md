#!/usr/bin/env python3

import rospy
import sys
import select
import termios
import tty

from geometry_msgs.msg import Twist


# ============================================================
# TURTLEBOT KEYBOARD TELEOPERATION
# ============================================================

# Keyboard controls
#
#        W
#        ↑
#   A ←  S  → D
#
# W = Forward
# S = Backward
# A = Turn Left
# D = Turn Right
#
# Q = Forward + Left
# E = Forward + Right
# Z = Backward + Left
# C = Backward + Right
#
# X = Stop
# ESC / Ctrl+C = Exit
#
# + = Increase speed
# - = Decrease speed
# ============================================================


# ------------------------------------------------------------
# Speed settings
# ------------------------------------------------------------

LINEAR_SPEED = 0.2
ANGULAR_SPEED = 0.5

MAX_LINEAR_SPEED = 0.5
MAX_ANGULAR_SPEED = 1.5

MIN_LINEAR_SPEED = 0.05
MIN_ANGULAR_SPEED = 0.1

SPEED_STEP = 0.05
ANGULAR_STEP = 0.1


# ------------------------------------------------------------
# Keyboard timeout
# ------------------------------------------------------------

KEY_TIMEOUT = 0.1


# ------------------------------------------------------------
# Save terminal settings
# ------------------------------------------------------------

settings = termios.tcgetattr(sys.stdin)


# ------------------------------------------------------------
# Get keyboard input
# ------------------------------------------------------------

def get_key():

    tty.setraw(sys.stdin.fileno())

    ready, _, _ = select.select(
        [sys.stdin],
        [],
        [],
        KEY_TIMEOUT
    )

    if ready:
        key = sys.stdin.read(1)
    else:
        key = ''

    termios.tcsetattr(
        sys.stdin,
        termios.TCSADRAIN,
        settings
    )

    return key


# ------------------------------------------------------------
# Stop robot
# ------------------------------------------------------------

def stop_robot(pub):

    twist = Twist()

    twist.linear.x = 0.0
    twist.linear.y = 0.0
    twist.linear.z = 0.0

    twist.angular.x = 0.0
    twist.angular.y = 0.0
    twist.angular.z = 0.0

    pub.publish(twist)


# ------------------------------------------------------------
# Main program
# ------------------------------------------------------------

def main():

    global LINEAR_SPEED
    global ANGULAR_SPEED

    # --------------------------------------------------------
    # Initialize ROS node
    # --------------------------------------------------------

    rospy.init_node(
        'turtlebot_keyboard_teleop',
        anonymous=True
    )

    # --------------------------------------------------------
    # Publisher
    # --------------------------------------------------------

    pub = rospy.Publisher(
        '/cmd_vel',
        Twist,
        queue_size=10
    )

    # --------------------------------------------------------
    # ROS loop rate
    # --------------------------------------------------------

    rate = rospy.Rate(10)

    # --------------------------------------------------------
    # Display instructions
    # --------------------------------------------------------

    print("")
    print("==============================================")
    print("       TURTLEBOT KEYBOARD TELEOPERATION")
    print("==============================================")
    print("")
    print("Movement:")
    print("")
    print("              W")
    print("              ↑")
    print("        A  ←  S  →  D")
    print("")
    print("W : Forward")
    print("S : Backward")
    print("A : Turn Left")
    print("D : Turn Right")
    print("")
    print("Q : Forward + Left")
    print("E : Forward + Right")
    print("Z : Backward + Left")
    print("C : Backward + Right")
    print("")
    print("X : STOP")
    print("")
    print("+ : Increase speed")
    print("- : Decrease speed")
    print("")
    print("CTRL+C : Exit")
    print("")
    print("----------------------------------------------")
    print("Linear speed  :", LINEAR_SPEED)
    print("Angular speed :", ANGULAR_SPEED)
    print("----------------------------------------------")
    print("")

    # --------------------------------------------------------
    # Twist message
    # --------------------------------------------------------

    twist = Twist()

    try:

        while not rospy.is_shutdown():

            # ------------------------------------------------
            # Get keyboard key
            # ------------------------------------------------

            key = get_key()

            # Reset velocities every cycle
            # This makes the robot stop when no key is pressed.

            twist.linear.x = 0.0
            twist.linear.y = 0.0
            twist.linear.z = 0.0

            twist.angular.x = 0.0
            twist.angular.y = 0.0
            twist.angular.z = 0.0


            # =================================================
            # FORWARD
            # =================================================

            if key == 'w' or key == 'W':

                twist.linear.x = LINEAR_SPEED

                print(
                    "FORWARD   | Linear:",
                    round(LINEAR_SPEED, 2)
                )


            # =================================================
            # BACKWARD
            # =================================================

            elif key == 's' or key == 'S':

                twist.linear.x = -LINEAR_SPEED

                print(
                    "BACKWARD  | Linear:",
                    round(LINEAR_SPEED, 2)
                )


            # =================================================
            # LEFT
            # =================================================

            elif key == 'a' or key == 'A':

                twist.angular.z = ANGULAR_SPEED

                print(
                    "LEFT      | Angular:",
                    round(ANGULAR_SPEED, 2)
                )


            # =================================================
            # RIGHT
            # =================================================

            elif key == 'd' or key == 'D':

                twist.angular.z = -ANGULAR_SPEED

                print(
                    "RIGHT     | Angular:",
                    round(ANGULAR_SPEED, 2)
                )


            # =================================================
            # FORWARD + LEFT
            # =================================================

            elif key == 'q' or key == 'Q':

                twist.linear.x = LINEAR_SPEED
                twist.angular.z = ANGULAR_SPEED

                print("FORWARD + LEFT")


            # =================================================
            # FORWARD + RIGHT
            # =================================================

            elif key == 'e' or key == 'E':

                twist.linear.x = LINEAR_SPEED
                twist.angular.z = -ANGULAR_SPEED

                print("FORWARD + RIGHT")


            # =================================================
            # BACKWARD + LEFT
            # =================================================

            elif key == 'z' or key == 'Z':

                twist.linear.x = -LINEAR_SPEED
                twist.angular.z = ANGULAR_SPEED

                print("BACKWARD + LEFT")


            # =================================================
            # BACKWARD + RIGHT
            # =================================================

            elif key == 'c' or key == 'C':

                twist.linear.x = -LINEAR_SPEED
                twist.angular.z = -ANGULAR_SPEED

                print("BACKWARD + RIGHT")


            # =================================================
            # STOP
            # =================================================

            elif key == 'x' or key == 'X':

                twist.linear.x = 0.0
                twist.angular.z = 0.0

                print("STOP")


            # =================================================
            # INCREASE SPEED
            # =================================================

            elif key == '+' or key == '=':

                LINEAR_SPEED += SPEED_STEP
                ANGULAR_SPEED += ANGULAR_STEP

                if LINEAR_SPEED > MAX_LINEAR_SPEED:
                    LINEAR_SPEED = MAX_LINEAR_SPEED

                if ANGULAR_SPEED > MAX_ANGULAR_SPEED:
                    ANGULAR_SPEED = MAX_ANGULAR_SPEED

                print("")
                print("Speed Increased")
                print(
                    "Linear speed  :",
                    round(LINEAR_SPEED, 2)
                )
                print(
                    "Angular speed :",
                    round(ANGULAR_SPEED, 2)
                )


            # =================================================
            # DECREASE SPEED
            # =================================================

            elif key == '-' or key == '_':

                LINEAR_SPEED -= SPEED_STEP
                ANGULAR_SPEED -= ANGULAR_STEP

                if LINEAR_SPEED < MIN_LINEAR_SPEED:
                    LINEAR_SPEED = MIN_LINEAR_SPEED

                if ANGULAR_SPEED < MIN_ANGULAR_SPEED:
                    ANGULAR_SPEED = MIN_ANGULAR_SPEED

                print("")
                print("Speed Decreased")
                print(
                    "Linear speed  :",
                    round(LINEAR_SPEED, 2)
                )
                print(
                    "Angular speed :",
                    round(ANGULAR_SPEED, 2)
                )


            # =================================================
            # UNKNOWN KEY
            # =================================================

            elif key != '':

                print(
                    "Unknown key:",
                    repr(key)
                )


            # ------------------------------------------------
            # Publish velocity
            # ------------------------------------------------

            pub.publish(twist)

            # ------------------------------------------------
            # Maintain ROS loop
            # ------------------------------------------------

            rate.sleep()


    except KeyboardInterrupt:

        print("")
        print("Keyboard interrupt received.")


    finally:

        # ----------------------------------------------------
        # ALWAYS STOP ROBOT BEFORE EXITING
        # ----------------------------------------------------

        stop_robot(pub)

        print("")
        print("Robot stopped safely.")
        print("Teleoperation terminated.")

        # Restore terminal
        termios.tcsetattr(
            sys.stdin,
            termios.TCSADRAIN,
            settings
        )


# ============================================================
# PROGRAM ENTRY POINT
# ============================================================

if __name__ == '__main__':

    main()
