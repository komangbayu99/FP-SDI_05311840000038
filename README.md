# FP-SDI_05311840000038

### Sistem Deteksi Untuk Mendeteksi Manusia dan Benda Mati

***
**Nama    : Bayu Trianayasa
NRP       : 05311840000038**

***
## Pengenalan

Deteksi dan klasifikasi objek dan juga manusia. Pada project ini akan berfungsi untuk mendeteksi adanya pencurian di suatu berangkas dengan cara mendeteksi manusia (pencuri) dan benda mati (Perhiasa-perhiasan/Tumpukan Uang)


## Dependensi

Python3,numpy, opencv 2.

### Download hal yang dibutuhkan

1. Let pip install darkflow globally in dev mode (still globally accessible, but changes to the code immediately take effect)
    ```
    pip install -e .
    ```

2. Install with pip globally
    ```
    pip install .
    ```

## Code Program

``import numpy as np
import cv2
import copy
import sys
import requests
``
- Download terlebih dahulu Libraries yang akan dipakai yang berada diatas di CMD ( Command Prompt )

``#np.set_printoptions(threshold=sys.maxsize)

cap=cv2.VideoCapture('footage.mp4')
``
- Pada variable cap akan Memanggil Library cv2 untuk mengimport/mengkoneksikan video ke kodingan 

``framecount = 1
``

- Fungsi framecount ini adalah untuk memberikan notifikasi kepada Bot Telegram setiap 50/1 Frame sekali biar tidak terlalu ngespam

``def telesend(text):
    url = "https://api.telegram.org/bot1590527198:AAGoYnQCFKi-Gi6ldLK5WsIHlfom6Yr1odU/sendMessage?chat_id=1265603130&parse_mode=Markdown&text=" + text
    requests.get(url)
``

- Diatas ini adalah kodingan untuk mengkoneksikan antara Bot Telegram dengan Kodingan projek ini

``dict_frame={}``

- Untuk Menyimpan Frame untuk Background update

``for s in range(0,50):
    cap.set(cv2.CAP_PROP_POS_FRAMES, s)
    _, frame = cap.read()
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    dict_frame[s] = copy.deepcopy(frame)
    frames=[]
for i in range(0, 50):``

- Ini Untuk menyimpan 50 frame pertama yang udah dianalisa

`` frames.append(dict_frame[i])
medianFrame = np.median(frames, axis=0).astype(dtype=np.uint8)


cv2.imshow('frame', medianFrame)

cv2.waitKey(0)

cap.set(cv2.CAP_PROP_POS_FRAMES, 49)
``
- Di section berfungsi untuk menunggu hasil analisa

``Textout = dict.fromkeys(['frame','n_objects','object','Area','Perimeter','Classification'])
text_file = open("Output.txt", "w")
text_file.write("Output\n")
text_file.close()
``
- Hasil output talah keluar berupa Frame, Object< Area, Perimeter, dan juga Klasifikasinya 

``ret = True

while (ret):
    # Read frame
    ret, frame = cap.read()
    if not ret:
        break

    #store the actual number of frame
    actual_frame = int(cap.get(cv2.CAP_PROP_POS_FRAMES))


    #convert frame to gray
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    #store the actual frame into the dictionary (used to compute the median)
    dict_frame[actual_frame] = copy.deepcopy(frame)



    # Calculate absolute difference of current frame and the median frame

    dframe = cv2.absdiff(frame, medianFrame)


    #filter to denoise
    dframe = cv2.GaussianBlur(dframe, (7, 7), 1)


    # Treshold to binarize

    th, dframe = cv2.threshold(dframe,thresh=0,maxval = 255, type=cv2.THRESH_BINARY + cv2.THRESH_TRIANGLE)



    # find all your connected components (white blobs in the image)
    nb_components, output, stats, centroids = cv2.connectedComponentsWithStats(dframe, connectivity=8)

    # connectedComponentswithStats yields every seperated component with information on each of them, such as size
    # the following part is just taking out the background which is also considered a component, but we don't want that.
    sizes = stats[1:, -1];
    nb_components = nb_components - 1



    # minimum size of blobs we want to keep (number of pixels)
    min_size = 300


    # for every component in the image, you keep it only if it's above min_size

    for i in range(0, nb_components):
        if sizes[i] >= min_size:
            dframe[output == i + 1] = 255
        else:
            dframe[output == i + 1] = 0

 ``
- Ini adalah kodingan looping untuk penganalisaan objek 

``
    # morphology operators to enhance quality of segmentation

    dframe = cv2.morphologyEx(dframe, cv2.MORPH_CLOSE, kernel=cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (3, 3)),iterations=12)
    dframe = cv2.morphologyEx(dframe, cv2.MORPH_OPEN,kernel =cv2.getStructuringElement(cv2.MORPH_RECT,(3,3)), iterations=2)
``
- kodingan ini membuktikan baha disini menggunakan metode Morphologi, Yang dimana disini Variable Dfram akan memanggil library cv2 dengan menggunakan/mengaktidkan metode morphologi yag dimana jika akan menganalisa suatu objek akan dianalasikan sesuai frame dari objek tersebut.

``
    #store the actual frame in the dictionary used to compute the median
    dict_frame[actual_frame][dframe == 255] = medianFrame[dframe == 255]
``
-Untuk menhitung mediannya

``
    # compute the new median frame considering the last 50 frames
    frames=[]
    for i in range(actual_frame - 49, actual_frame):
``

- Befungsi untuk menghitung nilai median baru berdasarkan 50 frame terakhir yang sudah di analisa 

``
        frames.append(dict_frame[i])
    medianFrame = np.median(frames, axis=0).astype(dtype=np.uint8)
``

- Hasil Frame telah keluar

``
    #connected components labelling
    contours, hierarchy = cv2.findContours(dframe, cv2.RETR_TREE, cv2.CHAIN_APPROX_NONE)
    frame = cv2.cvtColor(frame, cv2.COLOR_GRAY2RGB)
    frame_copy = copy.deepcopy(frame)
``
- Fungsi disini untuk Mengatur2 warna pada klasifikasi

``
    n_objects = len(contours)
    print("{:<10s}{:>4s}".format('frame', 'n_objects'))
    print("{:<10d}{:>4d}".format(int(actual_frame),n_objects))
    print()
    text_file = open("Output.txt", "a")
    text_file.write("\n")
    text_file.write("{:<10s}{:>4s}\n".format('frame', 'n_objects'))
    text_file.write("{:<10d}{:>4d}\n".format(int(actual_frame),n_objects))

    n=0
    for c in range(len(contours)):
``
-  untuk Objek akan di klasifikasian dengan durasi dibawah 10 detik dan diatas 4 detik

``
        Area = cv2.contourArea(contours[c])
        if Area > 4000:
            cv2.drawContours(frame, contours, c, (50, 0, 200), 2)
            Perimeter = cv2.arcLength(contours[c], True)
            print("{:<10s}{:>4s}{:>12s}{:>20s}".format('object', 'Area', 'Perimeter', 'Classification'))
            print("{:<10d}{:>4d}{:>6d}{:>20s}".format(c+1, int(Area) , int(Perimeter) , 'Person'))
            #text_file = open("Output.txt", "a")
            text_file.write("{:<10s}{:>4s}{:>12s}{:>20s}\n".format('object', 'Area', 'Perimeter', 'Classification'))
            text_file.write("{:<10d}{:>4d}{:>6d}{:>20s}\n".format(c+1, int(Area) , int(Perimeter) , 'Person'))
            coord=(contours[c][0][0][0],contours[c][0][0][1]-5)
            cv2.putText(frame, 'Person', coord, cv2.FONT_HERSHEY_SIMPLEX, 0.65, (50, 0, 200), 1, cv2.LINE_AA)
            framecount += 1
            if framecount % 50 == 0:
                telesend("Manusia Terdeteksi")

        else:
            n+=1
            cv2.drawContours(frame, contours, c, ((n%2)*180,50+n//2*100 , n%4*80), 2)
            Perimeter = cv2.arcLength(contours[c], True)
            edges = cv2.Canny(frame_copy, 0, 255)
            edges = cv2.dilate(edges, np.ones((3,3),np.uint8), iterations=1)

            con = np.vstack(contours[c]).squeeze()
``

- Pada Kodingan ini Berfungsi untuk menganalisa manusia yang dimana Jika Benda yang terdeteksi area > 4000 itu akan terperintah untuk mendeteksi, Jika sudah maka akan diteksi apakah itu manusia atau bukan bersdasarkan OBjek,area,perimeter dan juga akan di klasifikasikan. setelah itu kalo memang itu adalah manusia, sistem akan mendeteksi itu adalah manusia dan akan mengirim notifikasi ke BotTelegram "Manusia Terdeteksi"

``
            if np.sum((edges[con[:,1],con[:,0]]) == 255)/con.shape[0] > 0.5:
                print("{:<10s}{:>4s}{:>12s}{:>20s}".format('object', 'Area', 'Perimeter', 'Classification'))
                print("{:<10d}{:>4d}{:>6d}{:>20s}".format(c + 1, int(Area), int(Perimeter), 'Object_True'))
                text_file.write("{:<10s}{:>4s}{:>12s}{:>20s}\n".format('object', 'Area', 'Perimeter', 'Classification'))
                text_file.write("{:<10d}{:>4d}{:>6d}{:>20s}\n".format(c + 1, int(Area), int(Perimeter), 'Object_True'))
                coord = (contours[c][0][0][0], contours[c][0][0][1] - 5)
                cv2.putText(frame, 'Object_True', coord, cv2.FONT_HERSHEY_SIMPLEX, 0.65,
                            ((n % 2) * 180, 50 + n // 2 * 100, n % 4 * 80), 1, cv2.LINE_AA)
                framecount += 1
                if framecount % 20 == 0:
                    telesend("Benda Terdeteksi")

            else:
                print("{:<10s}{:>4s}{:>12s}{:>20s}".format('object', 'Area', 'Perimeter', 'Classification'))
                print("{:<10d}{:>4d}{:>6d}{:>20s}".format(c + 1, int(Area), int(Perimeter), 'Object_False'))
                text_file.write("{:<10s}{:>4s}{:>12s}{:>20s}\n".format('object', 'Area', 'Perimeter', 'Classification'))
                text_file.write("{:<10d}{:>4d}{:>6d}{:>20s}\n".format(c + 1, int(Area), int(Perimeter), 'Object_False'))
                coord = (contours[c][0][0][0], contours[c][0][0][1] - 5)
                cv2.putText(frame, 'Object_False', coord, cv2.FONT_HERSHEY_SIMPLEX, 0.65,
                            ((n % 2) * 180, 50 + n // 2 * 100, n % 4 * 80), 1, cv2.LINE_AA)
``
- fungsi ini adalah untuk mendeteksi objek mati, jika itu emang terdeteksi benda mati maka sistem akan mengirim notifikasi ke bot telegram "Benda Terdeteksi"

``
    text_file.close()
    # Display image


    cv2.imshow('frame', frame)

    cv2.waitKey(40)


    print()
    print()
    print()
    print()
    print()

# Release video object
cap.release()

# Destroy all windows
cv2.destroyAllWindows()
``
