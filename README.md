<p align="center">
    <img src="PT_IUS_wname.png" width="50%" height="100%">
</p>

# Automatic Reject System for Blister Defect
## **Overview**
The Automatic Reject System for Blister Defect project aims to enhance the efficiency and accuracy of quality control processes in blister packaging manufacturing. Blister packaging is widely used in various industries for packaging medicines, electronics, and other sensitive products. Ensuring the integrity of blister packs is crucial to maintain product quality and safety. This project focuses on developing an automated system to identify and reject blister packs with defects, streamlining the quality control process. We made this system with so many components, **includings**:
- **Python** (Base language)
- **AI/Machine Learning** (YOLO v8 by Ultralytics)
- **PyQt5** (GUI to make ease the interaction)
- **OpenCV** (Image processing)
- **Arduino Uno** (Connected with proximity sensor and servo motor, both for Rejector)
- **Talos** Executor (PT. IUS framework to communicate with Arduino in Python)
- **Webcam** (Logi C922 Pro 60Fps)
- **Computer/Mini PC** (Compute machine) 

## **How to install:**
1. Make sure you have **Python 3.9** installed  
Notes: If you use Computer with processor under Intel i3 or any processor without **avx instructions**, you have to install **Python 3.8**
2. **(OPTIONAL)** Inside terminal, create venv by running 
```bash
python -m venv venv
``` 
you can change venv with any names you desired. 
Then, activate venv by running 
```bash
venv/Scripts/activate
```
3. Install all module required
```bash
pip install -r requirements.txt
```
4. For **OCR requirements**, if you use device with processor that not supporting avx instruction, do these steps:
    1. Uninstall current Paddle library
    ```bash
    pip uninstall paddlepaddle
    ```
    2. Run the following command to re-install paddle library. **Important: It only available at python 3.8**
    ```bash
    python -m pip download paddlepaddle==2.4.2 -f https://www.paddlepaddle.org.cn/whl/linux/mkl/noavx/stable.html --no-index --no-deps
    ```    
    3. Then, you install new Paddle package   
    ```bash
    python -m pip install paddlepaddle-2.4.2-cp38-cp38-win_amd64.whl
    ```

5. **(MANUAL)** Start Program
    1. Run OCR system
    ```bash
    python OCR.py
    ```
    2. In different terminal, run
    ```bash
    python Apps.py
    ```

6. **(SHORTCUT)** Start Program
    1. Do some configuration in **run_scripts.bat** by editing all the PATHs depending on the repo directory you are using
    2. Now, you can double-click **run_scripts.bat** and run the program
    3. Optionally, you can make shortcut **run_scripts.bat** to Desktop so user can use it more easily

## **How to Use:**
### Preparation:
1. Make sure the installation process is complete without any errors
2. Make sure The Rejector part is done and functional, **Talos** must be installed to all Arduino Boards
3. Change the port number in TalosProxServ.py depends on the computer itself (Win+X then open device manager). Arduino that connect to servo is **Rejector** and proximity sensor is **Counter**
    ```python
                                #Ex.
    c1 = Rejector(bySerial(port="COM6"))
    c2 = Counter(bySerial(port="COM11"))
    ```
4. Connect the usb cable that also connected to Arduino
5. Start The Application
### Using Application:
1. First, you need to choose how much blister inside one frame by pressing **(3)** or **(4)** button
2. Count the distance or how many blister from Camera Point to Rejector, then adjust by simply press **(-)** or **(+)** button
3. Input the batch number, Edition Date, and Price on the input boxes. 
4. Press **(START)** button
5. If you are planning to change the amount of blister or the input texts, make sure to press **(STOP)** first
6. If somehow the Line Counting is messed up, you can just press **(CALIBRATE)**
### Camera Control:
The application provides some keyboard control of the camera view, this is because sometimes the position of the blister itself is different for each type.
<div style="display: flex;">
    <div style="flex: 50%; padding-right: 10px;">
        <h4>Defect Detection Camera:</h4>
        <table>
            <thead>
                <tr>
                    <th>KEY</th>
                    <th>Function</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><strong>W</strong></td>
                    <td>Slide camera view up</td>
                </tr>
                <tr>
                    <td><strong>A</strong></td>
                    <td>Slide camera view down</td>
                </tr>
                <tr>
                    <td><strong>S</strong></td>
                    <td>Narrow camera view</td>
                </tr>
                <tr>
                    <td><strong>D</strong></td>
                    <td>Expand camera view</td>
                </tr>
            </tbody>
        </table>
    </div>
    <div style="flex: 50%; padding-left: 10px;">
        <h4>OCR Camera:</h4>
        <table>
            <thead>
                <tr>
                    <th>KEY</th>
                    <th>Function</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><strong>I</strong></td>
                    <td>Slide camera view up</td>
                </tr>
                <tr>
                    <td><strong>K</strong></td>
                    <td>Slide camera view down</td>
                </tr>
                <tr>
                    <td><strong>O</strong></td>
                    <td>Bringing the focus point closer</td>
                </tr>
                <tr>
                    <td><strong>P</strong></td>
                    <td>Move the focus point away</td>
                </tr>
            </tbody>
        </table>
    </div>
</div>

# Program Code Flow
## Blister Defect Rejector System
This section will explains about how the code works (Apps.py), how we operate arduino (TalosProxServ.py), and how we communicate between apps and the arduino. We will seperate into some sub-base to make it easier to understand.

### Machine Learning
1. Our AI system using YOLO based computer vision, here we choose YOLOv8 by Ultralytics to determine which part of blister is missing pills and which not. Inside this apps we used the "Nano" structure because we think that the task is pretty straight forward and dont really needs a complicated model. Beside that, we also aim to have a model that fast enough to predict so it can be more efficient.
2. To see clear you can see starting from this line of code [processing YOLOv8](Apps.py#L554). In this [code](Apps.py#L554) we check if a frame is stopped after moving using OpenCV, and if it is stopped then we catch and throw it into the model and do predictions. From there we process the result by doing ```data = results[0].boxes.data```, this line of code stores about bounding box informations and labels for each prediction. We also use these information to perform some calculation summary of total blister by checking if inside a frame contains a filled pills or missed fill pills (needs improvement).

### Main Application Layout
Our application basically only has one main window for its operation, we built using PyQt5. As you can see, we create our first main layout starting from around **Line-100** until **Line-333**. We also create some function/methods to make the layout more interactive and informative.
1. [draw()](Apps.py#L424) is used to draw lines for separating each blister and write Title in frame
2. [toggle_camera()](Apps.py#L649) button that can control all behaviour and what it does when START/STOP button pressed
3. [update_text()](Apps.py#L699) is function that will update all text (counting blister section) each time its called
4. [conf_button()](Apps.py#L710) this function control almost all of button behaviour which button must be pressed first or which button is not supposed to pressed (mostly using methods .setEnabled(), you can improve this by applying diff color each case)
5. [CircleWidget](Apps.py#L710) class which used to make a circle of indicator (RED is when defect detected, GREEN is no defect, YELLOW is when system paused)

### Arduino (TalosProxServ.py)
I need to clarify first that our apps comunicate with arduino using dictionary named **self.defect_dict** where [key] = which **line** of blister is defect and [value] = stored which part in those line is defect (because in one line there might be 3/4 blister) which in result looked like this
```python
defect_dict = {
    line: [part1, part2],
    #Ex.
    23: [1, 4],
}
```
Lets talking about code flows:
1. We create a [init_arduino()](Apps.py#L394) (MUST RUN IN OTHER THREAD) that create object stored in var/attr. named **self.arduino_rejector** which also init the Talos Arduino and start processing, here i also set the params about how much blister inside one line and the defect_dict itself.
2. If we look in [start_process()](TalosProxServ.py#L113) methods we can see that we run **self.process_proxi()** in other thread to count how much blister passed and in which line the proxi is detecting. The logic is if a blister detected by proxi it counts on **unit_count** (+=1 each time blister passed) which also line counting but only for each part, then only if all part is passed then the **line_count** is +=1. 
3. By using the information from the **defect_dict** and the from proximity sensor the [process_rejector()](TalosProxServ.py#L105) start seeing wether the line passed proximity sensor is counted as line that contains defect or not.  
  
This system still needs an improvement especially in counting logic, feel free to improvise and change the logic.

## OCR:
This section explains about the file content of OCR.py. For your information, the program mainly use asyncronous method for fetching data image and predicting OCR model. The methods will be explained in above steps.

1. The function **read_file_to_queue** will do fetching image data from pickle file that stored from program Apps. and put to queue of list data that will be predicted by OCR model.  
2. The function **perform_ocr** returns the result of predicting by OCR model. 
3. The function **main** processes **read_file_to_queue** continously. 
4. The function **process_ocr** get the image data from queue and stored to **perform_ocr**. The result will be sent to pickle file.
5. The **run_tasks** create the task for running two functions (**main** and **process_ocr**) simultaneously.

# Developers
## Programmers Note
This system in only in first stage of development, would be a pleasure for us if you continue to develop our application so that it becomes a much more advanced application. We will try to list the things that might be developed into this system/application to be better such as:
- The model needs an improvement of seeing new type of medicine/blister/pills because in real case the variety is uncountable. Our advice is you can enlarge the dataset and then try to add the complexity of the model itself.
- The model may needs to see other things rather than just filled and missed pills. Such as seepertaing each blister, seeing cracked pills, seeing double filled pills, even better if the model can see a blister pack is broken.
- User Interface might also needs improvement, you can add more information or maybe change some color depends on the case itself.
- The counting logic in this system is not accurate. We're not saying that this is bad, it's just that the logic we've implemented in the app is very easy to mess up with even the slightest movement. We hope you can make better logic and more stable in real usage.
- Finally, there is the matter of code efficiency where our program still has some lagging such as when first opening the application, when first doing predictions, and doing ocr predictions where for this case we even had to split the terminal to avoid lagging.   

In the end, we hope that the application that we developed in this early stage can grow far enough to be used by many companies, making it easier to solve problems in the factory production area. We are also open to future developers if you want to ask a few questions or ask for suggestions about this system and application.
## Programmer:
| Name                     | University                   |
|--------------------------|------------------------------|
| Dimas Irfan Sya'roni Zaki| Universitas Dian Nuswantoro  |
| Yohannes Alexander Sinaga| Universitas Airlangga        |

## Mechanic:
| Name                     | University                   |
|--------------------------|------------------------------|
| Fathur Rahman Sigara     | Universitas Hasanuddin       |