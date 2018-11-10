# mandala by highLevel

# Basic Animation Framework

from tkinter import *
import math


####################################
def getPolarCoordinates(cx, cy, n, x, y):
    dx = cx - x
    dy = cy -y
    deltaTheta = 2 * math.pi / n
    r = (dx ** 2 + dy**2) ** 0.5
    angle = math.atan(dy / dx)
    quadrant = getQuadrant(cx, cy, x, y)
    bigTheta = quadrant * math.pi / 2 + angle
    pieSlice = getPieSlice(bigTheta, n)
    pieSliceTheta = deltaTheta * pieSlice
    theta = bigTheta - pieSliceTheta
    return (r, theta)
    

def getQuadrant(cx, cy, x, y):
    if x > cx:
        if y > cy:
            return 3
        return 0
    else:
        if y > cy:
            return 2
        return 1
        
def getPieSlice(theta, numSlices):
    dA=2*math.pi/numSlices
    slice=theta//dA
    return int(slice)

####################################
# customize these functions
####################################

def init(data):
    # load data.xyz as appropriate
    data.lines = [] #eventual 3D list containing all the lines (lists of lineLsts of lines of tuples)
    data.currLine = [] #1D list containing tuples
    data.cx = data.width/2
    data.cy = data.height/2
    data.numSlices = 6 #number of pie slices
    data.maxLimit = 100
    data.limit = 0

def mousePressed(event, data):
    # use event.x and event.y
    if data.limit < data.maxLimit:
        r, offset = getPolarCoordinates(data.cx, data.cy, data.numSlices, event.x, event.y)
        data.currLine.append((r, offset))
        data.limit += 1
        print(data.limit)
    elif data.limit != 1000:
        print("Stop")
        data.limit = 1000
        commitCurrLine(data)

def mousePressReleased(event, data):
    commitCurrLine(data)

def keyPressed(event, data):
    # use event.char and event.keysym
    pass

def redrawAll(canvas, data):
    # draw in canvas
    for lineLst in data.lines:
        for line in lineLst:
            canvas.create_line(line, smooth="true")
    if len(data.currLine) >= 2:
        tempLineLst = convertCurrLine(data)
        for line in tempLineLst:
            canvas.create_line(line, smooth="true")

#commits the current line to the entire data.lines 3d list
def commitCurrLine(data):
    if len(data.currLine) >= 2:
        dA = 2*math.pi/data.numSlices
        tempLine = convertCurrLine(data)
        data.lines.append(tempLine)
        data.currLine = []

#converts current line to six lines with tuple values listing x and y values
def convertCurrLine(data):
    #if len(data.currLine) >= 2?
    dA = 2*math.pi/data.numSlices
    tempLine = []
    for currSlice in range(data.numSlices):
        sliceAngle = currSlice*dA
        line1 = []
        line2 = []
        for tup in data.currLine:
            r, offset = tup
            x1, y1 = convertToCartesian(data.cx, data.cy, r, sliceAngle + offset)
            line1.append((x1, y1))
            x2, y2 = convertToCartesian(data.cx, data.cy, r, sliceAngle - offset)
            line2.append((x2, y2))
        tempLine.append(line1)
        tempLine.append(line2)
    return tempLine #2D list of lists of tuples with points of a line

def convertToCartesian(cx, cy, r, theta):
    x = cx + r*math.cos(theta)
    y = cy - r*math.sin(theta)
    return (x, y)

####################################
# use the run function as-is
####################################

def run(width=300, height=300):
    def redrawAllWrapper(canvas, data):
        canvas.delete(ALL)
        canvas.create_rectangle(0, 0, data.width, data.height,
                                fill='white', width=0)
        redrawAll(canvas, data)
        canvas.update()    

    def mousePressedWrapper(event, canvas, data):
        mousePressed(event, data)
        redrawAllWrapper(canvas, data)
    
    def mousePressReleasedWrapper(event, canvas, data):
        mousePressReleased(event, data)
        redrawAllWrapper(canvas, data)

    def keyPressedWrapper(event, canvas, data):
        keyPressed(event, data)
        redrawAllWrapper(canvas, data)

    # Set up data and call init
    class Struct(object): pass
    data = Struct()
    data.width = width
    data.height = height
    root = Tk()
    root.resizable(width=False, height=False) # prevents resizing window
    init(data)
    # create the root and the canvas
    canvas = Canvas(root, width=data.width, height=data.height)
    canvas.configure(bd=0, highlightthickness=0)
    canvas.pack()
    # set up events
    root.bind("<B1-Motion>", lambda event: #changed; tracks held-down motion
                            mousePressedWrapper(event, canvas, data))
    root.bind("<B1-ButtonRelease>", lambda event: 
                            mousePressReleasedWrapper(event, canvas, data))
    root.bind("<Key>", lambda event:
                            keyPressedWrapper(event, canvas, data))
    redrawAll(canvas, data)
    # and launch the app
    root.mainloop()  # blocks until window is closed
    print("bye!")

run(800, 800)
