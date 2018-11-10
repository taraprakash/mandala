# mandala by highLevel

# Basic Animation Framework

from tkinter import *
import math


####################################
def getPolarCoordinates(cx, cy, n, x, y):
    dx = cx - x
    dy = cy -y
    deltaTheta = 2 * math.pi / n
    quadrant = getQuadrant(cx, cy, x, y)
    if dx == 0:
        r = dy
        if quadrant == 1:
            theta = math.pi / 2
        else:
            theta = 3 * math.pi / 2
    else:
        angle = math.atan(dy / dx)
        bigTheta = quadrant * math.pi / 2 + angle
        pieSlice = getPieSlice(bigTheta, n)
        pieSliceTheta = deltaTheta * pieSlice
        theta = bigTheta - pieSliceTheta
        r = (dx ** 2 + dy**2) ** 0.5
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
    data.undoLst = []
    data.YTopMargin=(70/490)*data.height
    data.YBottomMargin=(40/490)*data.height
    data.XMargin=(20/440)*data.width
    data.cx = data.width/2
    data.cy = (data.height+data.YTopMargin-data.YBottomMargin)/2
    data.numSlices = 6 #number of pie slices
    data.fontSize=(15/70)*data.YTopMargin
    data.font="bold "+str(int(data.fontSize))
    data.maxLimit = 100
    data.limit = 0
    data.mode = "startScreen"
    data.colors = ["black", "red4", "pale violet red", "sea green", "DeepSkyBlue4"]
    data.currColor = 0

def mousePressed(event, data):
    # use event.x and event.y
    if data.mode == "gameScreen":
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
    if data.mode == "gameScreen":
        commitCurrLine(data)
        
def keyPressed(event, data):
    # use event.char and event.keysym
    if data.mode == "startScreen":
        if event.keysym == "s":
            data.mode = "gameScreen"
        elif event.keysym.isdigit() and 4 <= int(event.keysym) <= 8:
            data.numSlices = int(event.keysym)
        elif event.keysym == "Right":
            data.currColor += 1
            if data.currColor >= len(data.colors):
                data.currColor = len(data.colors) - 1
        elif event.keysym == "Left":
            data.currColor -= 1
            if data.currColor < 0:
                data.currColor = 0
    elif data.mode == "gameScreen":
        if event.keysym == "b":
            init(data)
        elif event.keysym == "u" and len(data.lines) > 0:
            data.undoLst.append(data.lines.pop())
        elif event.keysym == "r" and len(data.undoLst) > 0:
            data.lines.append(data.undoLst.pop())
    
 
def drawProgress(canvas,data):
    startX=(data.XMargin*6)
    barLen=data.width-data.XMargin-startX
    barThick=data.YBottomMargin//2
    if data.limit<=100:
        lim=data.limit
        done=False
    else:
        lim=100
        done=True
    progLen=barLen*(lim/100)
    startY=data.height-((barThick/2)*3)
    endY=data.height-(barThick/2)
    canvas.create_rectangle(startX, startY, data.width-data.XMargin, endY, fill="white", width=2)
    if done:
        canvas.create_text(data.XMargin*2, startY, text="Done!", anchor=NW, fill="black", font=data.font)
        canvas.create_rectangle(startX, startY, startX+progLen, endY, fill="gray20")
    else:
        canvas.create_text(data.XMargin, startY, text="Progress:", anchor=NW, fill="black", font=data.font)
        canvas.create_rectangle(startX, startY, startX+progLen, endY, fill="gray47")
        
def drawGameScreen(canvas, data):
    for lineLst in data.lines:
        for line in lineLst:
            canvas.create_line(line, width = 2, smooth="true", fill = data.colors[data.currColor])
    if len(data.currLine) >= 2:
        tempLineLst = convertCurrLine(data)
        for line in tempLineLst:
            canvas.create_line(line, width = 2, smooth="true", fill = data.colors[data.currColor])
    canvas.create_rectangle(0,0,data.width, data.YTopMargin, fill="lightSteelBlue", outline="lightSteelBlue")
    canvas.create_rectangle(0,0,data.XMargin, data.height, fill="lightSteelBlue", outline="lightSteelBlue")
    canvas.create_rectangle(data.width-data.XMargin, 0, data.width, data.height, fill="lightSteelBlue", outline="lightSteelBlue")
    canvas.create_rectangle(0, data.height-data.YBottomMargin, data.width, data.height, fill="lightSteelBlue", outline="lightSteelBlue")
    canvas.create_line(data.XMargin, data.YTopMargin, data.XMargin, data.height-data.YBottomMargin, width=2)
    canvas.create_line(data.width-data.XMargin, data.YTopMargin, data.width-data.XMargin, data.height-data.YBottomMargin, width=2)
    canvas.create_line(data.XMargin, data.YTopMargin, data.width-data.XMargin, data.YTopMargin, width=2)
    canvas.create_line(data.XMargin, data.height-data.YBottomMargin, data.width-data.XMargin, data.height-data.YBottomMargin, width=2)
    canvas.create_text(data.width//2, data.YTopMargin//4, text="Click and drag to draw, press \"u\" to undo", font=data.font)
    canvas.create_text(data.width//2, data.YTopMargin*3//4, text="press \"r\" to restart", font=data.font)
    drawProgress(canvas, data)
    
def drawStartScreen(canvas, data):
    canvas.create_rectangle(0, 0, data.width, data.height, fill = "lightSteelBlue")
    canvas.create_text(data.cx, data.height//4, text = "Let\'s Make A \n   Mandala!",
                        font = "Times  50")
    canvas.create_text(data.width//3, data.cy, text = "Type a number \n between 4 and 8 \n to choose your \n number of slices", font = "Times 15")
    #canvas.create_oval(2 * data.width//3 - 40, data.cy - 40,
                       #2 * data.width//3 + 40, data.cy + 40)
    canvas.create_rectangle(2 * data.width//3 - 20, data.cy - 20, 
                            2 * data.width//3 + 20, data.cy + 20,
                            fill = "white")
    canvas.create_text(2 * data.width//3, data.cy, text = str(data.numSlices), font = "Times 20")
    canvas.create_text(data.cx, data.height - 45, text = "Press s to start!", font = "Times 20")
    canvas.create_text(data.width//4, data.height - 110, text = "Use arrow keys \n to pick a color", font = "Times 15")
    barWidth = 200
    canvas.create_rectangle(data.cx, data.height - 130, 
                            data.cx + barWidth, data.height - 80)
    blockWidth = barWidth // len(data.colors)
    for i in range(len(data.colors)):
        color = data.colors[i]
        print(color)
        canvas.create_rectangle(data.cx + blockWidth * i, data.height - 130, 
                            data.cx + blockWidth * (i + 1), data.height - 80, fill = color)
    print(data.currColor)
    canvas.create_rectangle(data.cx + data.currColor * blockWidth, data.height - 130,
                            data.cx + blockWidth * (data.currColor + 1), data.height - 80, width = 5, fill = None)
                            
    
def redrawAll(canvas, data):
    # draw in canvas
    if data.mode == "gameScreen":
        drawGameScreen(canvas, data)
    elif data.mode == "startScreen":
        drawStartScreen(canvas, data)
def redrawAll(canvas, data):
    # draw in canvas
    if data.mode == "gameScreen":
        drawGameScreen(canvas, data)
    elif data.mode == "startScreen":
        drawStartScreen(canvas, data)

#commits the current line to the entire data.lines 3d list
def commitCurrLine(data):
    if len(data.currLine) >= 2:
        dA = 2*math.pi/data.numSlices
        tempLine = convertCurrLine(data)
        data.lines.append(tempLine)
        data.currLine = []
    if len(data.undoLst) != 0:
        data.undoLst = []

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

#converts polar coordinates to cartesian
def convertToCartesian(cx, cy, r, theta):
    x = cx + r*math.cos(theta)
    y = cy - r*math.sin(theta)
    return (x, y)

####################################
# use the run function as-is
####################################

def run(width=440, height=510):
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

run(440, 490)
