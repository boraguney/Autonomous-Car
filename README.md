# Autonomous-Car
I add some codes that were used in the preparation of a mini autonomous car.
#


import cv2
import numpy as np
import glob

label_names = ["Girisi_olmayan_yol",
               "Sola_mecburi_yon",
               "Saga_mecburi_yon",
               "Soldan_gidiniz",
               "ileri_ve_sag_mecburi_yon",
               "Yaya_gecidi",
               "Gevsek_malzemeli_zemin",
               "Yolda_calisma",
               "Park_yeri"]

drawing = False
point1 = ()
point2 = ()


def nothing(x):
    pass


def get_img_names():
    imgs = glob.glob("./bora/*.jpg")
    return imgs


def setControlPanel(label_names):
    cv2.namedWindow("control")
    # create switch for ON/OFF functionality
    for name in label_names:
        cv2.createTrackbar(name, 'control',0,1,nothing)    


def setImageWindow():
    cv2.namedWindow("Frame")
    cv2.setMouseCallback("Frame", mouse_drawing)


def mouse_drawing(event, x, y, flags, params):
    global point1, point2, drawing
    if event == cv2.EVENT_LBUTTONDOWN:
        if drawing is False:
            drawing = True
            point1 = (x, y)
        else:
            drawing = False
 
    elif event == cv2.EVENT_MOUSEMOVE:
        if drawing is True:
            point2 = (x, y)


def showOldLabels(image, labels):
    for label in labels:
        cv2.rectangle(image, label[1], label[2], (0,0,255),2) 
    return image


def showControlPanel(label_names):
    trackbars = []
    for name in label_names:
        a = cv2.getTrackbarPos(name, 'control')
        trackbars.append(a)

    return trackbars
 

def run():
    print("Welcome to Labelling Tool")
    labels = []
    setControlPanel(label_names)
    setImageWindow()

    img_names = get_img_names()

    for img_name in img_names:
        with open("labels.txt", "r") as file:
            line = file.readline()
            count = 1
            break_it = False
            while line:
                if line[:17] == img_name:
                    print(line[:17])
                    break_it = True 
                line = file.readline()
                count += 1
            if break_it:
                print("breaking")
                break_it = False
                continue

        while True:
            trackbars = showControlPanel(label_names)

            frame = cv2.imread(img_name)

            frame = showOldLabels(frame, labels)
        
            if point1 and point2:
                cv2.rectangle(frame, point1, point2, (0, 255, 0), 2)

            cv2.imshow("Frame", frame)

            key = cv2.waitKey(1)
            if key == 27:
                with open("labels.txt","a") as file:
                    for label in labels:
                        file.write(img_name + ", " +str(label) + "\n")
                labels = []
                break
            if key == 32:
                sign = ""
                
                if trackbars[np.argmax(trackbars)] != 0:
                    sign = label_names[np.argmax(trackbars)]
                else:
                    print("You must choose a sign")
                    continue

                sign_array = [sign, point1, point2]
                labels.append(sign_array)
                print(labels)
            if key == 120:
                labels = []
    
    cv2.destroyAllWindows()

run()
