# Project-Report-Object-Recognition-using-HuskyLens
Project Report: Object Recognition using HuskyLens
1. Introduction HuskyLens is an AI-powered vision sensor designed for machine learning applications. It can recognize objects, track them, and classify them with minimal programming effort. In this project, we utilized HuskyLens for Object Recognition, allowing it to detect and identify objects in real-time.
2. Objectives
•	Implement Object Recognition using HuskyLens.
•	Integrate the sensor with an Arduino board.
•	Process and display detection results.
•	Control a robotic system based on object recognition results.
3. Hardware and Software Used
•	Hardware: 
o	HuskyLens AI Camera
o	Arduino Board (Uno, Mega, or Leonardo)
•	Software: 
o	Arduino IDE
o	HuskyLens Library
o	C++ for Arduino programming
4. Implementation Steps
1.	Setup HuskyLens: 
o	Connect HuskyLens to the Arduino via I2C communication.
o	Ensure that HuskyLens is in Object Recognition Mode.
o	Train the sensor to recognize specific objects.
2.	Write the Arduino Code: 
o	Initialize HuskyLens in I2C mode.
o	Request object recognition data.
o	Control a robotic system based on detected objects.
3.	Testing and Debugging: 
o	Validate the detection accuracy of HuskyLens.
o	Adjust parameters like width levels and motion control.
o	Display recognition data using Serial Monitor.
5. Results
•	The system successfully detected and recognized trained objects.
•	The robotic system responded accurately based on object detection.
•	The project demonstrated real-time object tracking and movement control.
6. Challenges and Solutions
•	Detection Accuracy: Sometimes, the recognition was not precise. This was improved by adjusting training conditions and ensuring proper lighting.
•	I2C Communication Errors: These were resolved by checking wiring connections and setting the correct protocol type in HuskyLens settings.
7. Conclusion This project successfully implemented Object Recognition using HuskyLens, demonstrating its capability in AI-driven object detection. The results show potential applications in robotics, automation, and AI-based vision systems.
________________________________________
Code:
/***************************************************
 HUSKYLENS An Easy-to-use AI Machine Vision Sensor
 <https://www.dfrobot.com/product-1922.html>
 
 ***************************************************
 This example shows how to play with object tracking.
 
 Created 2020-03-13
 By [Angelo qiao](Angelo.qiao@dfrobot.com)
 
 GNU Lesser General Public License.
 See <http://www.gnu.org/licenses/> for details.
 All above must be included in any redistribution
 ****************************************************/

/***********Notice and Trouble shooting***************
 1.Connection and Diagram can be found here
 <https://wiki.dfrobot.com/HUSKYLENS_V1.0_SKU_SEN0305_SEN0336#target_23>
 2.This code is tested on Arduino Uno, Leonardo, Mega boards.
 ****************************************************/


#include "HUSKYLENS.h"
#include "DFMobile.h"

DFMobile Robot (7,6,4,5);     // initiate the Motor pin

HUSKYLENS huskylens;
//HUSKYLENS green line >> SDA; blue line >> SCL

void setup() {
    Serial.begin(115200);
    Robot.Direction (HIGH, LOW);  // initiate the positive direction  

    Wire.begin();
    while (!huskylens.begin(Wire))
    {
        Serial.println(F("Begin failed!"));
        Serial.println(F("1.Please recheck the \"Protocol Type\" in HUSKYLENS (General Settings>>Protocol Type>>I2C)"));
        Serial.println(F("2.Please recheck the connection."));
        delay(100);
    }

    huskylens.writeAlgorithm(ALGORITHM_OBJECT_TRACKING); //Switch the algorithm to object tracking.

    // while (true)
    // {Robot.Speed (200-50,200+50);
    // delay(2000);
    // Robot.Speed (0,0);
    // delay(2000);
    // Robot.Speed (200,200);
    // delay(2000);
    // Robot.Speed (0,0);
    // delay(2000);
    // Robot.Speed (200+50,200-50);
    // delay(2000);
    // Robot.Speed (0,0);
    // delay(2000);}

}

int widthLevel = 50;

int xLeft = 160-40;
int xRight = 160+40;

bool isTurning = false;
bool isTurningLeft = true;

bool isInside(int value, int min, int max){
    return (value >= min && value <= max);
}

void printResult(HUSKYLENSResult result);

void loop() {
    int32_t error;

    int left = 0, right = 0;

    if (!huskylens.request()) Serial.println(F("Fail to request objects from HUSKYLENS!"));
    else if(!huskylens.isLearned()) {Serial.println(F("Object not learned!")); Robot.Speed (0,0);}
    else if(!huskylens.available()) Serial.println(F("Object disappeared!"));
    else
    {
        HUSKYLENSResult result = huskylens.read();

        if (result.width < widthLevel){
            widthLevel = 65;
            if (isInside(result.xCenter, 0, xLeft)){
                if (isTurningLeft){
                    if (!isTurning){
                        Robot.Speed (200-50,200+50);
                    }
                }
                else{
                    if (isTurning){
                        isTurning = false;
                        isTurningLeft = !isTurningLeft;
                    }
                    Robot.Speed (200-50,200+50);
                }
            }
            else if (isInside(result.xCenter, xLeft, xRight)){
                if (isTurning){
                    isTurning = false;
                    isTurningLeft = !isTurningLeft;
                }
                Robot.Speed (200,200);
            }
            else if (isInside(result.xCenter, xRight, 320)){
                if (isTurningLeft){
                    if (isTurning){
                        isTurning = false;
                        isTurningLeft = !isTurningLeft;
                    }
                    Robot.Speed (200+50,200-50);
                }
                else{
                    if (!isTurning){
                        Robot.Speed (200+50,200-50);
                    }
                }
            }
        }
        else
        {
            widthLevel = 55;
            isTurning = true;
            if (isTurningLeft){
                Robot.Speed (0,200);
            }
            else{
                Robot.Speed (200,0);
            }
        }
        printResult(result);
    }
}

void printResult(HUSKYLENSResult result){
    if (result.command == COMMAND_RETURN_BLOCK){
        Serial.println(String()+F("Block:xCenter=")+result.xCenter+F(",yCenter=")+result.yCenter+F(",width=")+result.width+F(",height=")+result.height+F(",ID=")+result.ID);
    }
    else if (result.command == COMMAND_RETURN_ARROW){
        Serial.println(String()+F("Arrow:xOrigin=")+result.xOrigin+F(",yOrigin=")+result.yOrigin+F(",xTarget=")+result.xTarget+F(",yTarget=")+result.yTarget+F(",ID=")+result.ID);
    }
    else{
        Serial.println("Object unknown!");
    }
}







