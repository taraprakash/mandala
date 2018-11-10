# mandala
highLevel

# Basic Animation Framework

from tkinter import *

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
    

def getQuadrant(data, x, y):
    elif x > data.cx:
        if y > data.cy:
            return 3
        return 0
    else:
        if y > data.cy:
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
    pass

def mousePressed(event, data):
    # use event.x and event.y
    pass

def keyPressed(event, data):
    # use event.char and event.keysym
    pass

def redrawAll(canvas, data):
    # draw in canvas
    for lineLst in data.lines:
        for line in lineLst:
            canvas.create_line(line, smooth="true")
    if len(data.currLine) >= 2:
        tempLineLst = convertCurrLine(data.currLine, data.numSlices)
        for line in tempLineLst:
            canvas.create_line(line, smooth="true")

#commits the current line to the entire data.lines 3d list
def commitCurrLine(data):
    #if len(data.currLine) >= 2?
    dA = 2*math.pi/data.numSlices
    tempLine = convertCurrLine(data.currLine, data.numSlices)
    data.lines.append(tempLine)
    data.currLine = []

#converts current line to six lines with tuple values listing x and y values
def convertCurrLine(data):
    #if len(data.currLine) >= 2?
    dA = 2*math.pi/data.numSlices
    tempLine = []
    for currSlice in range(data.numSlices):
        sliceAngle = data.currSlice*dA
        line1 = []
        line2 = []
        for tup in line:
            r, offset = tup
            x1, y1 = convertToCartesian(data.cx, data.cy, r, sliceAngle + offset)
            line1.append(x1, y1)
            x2, y2 = convertToCartesian(data.cx, data.cy, r, sliceAngle - offset)
            line2.append(x2, y2)
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
    
    
    def eventInfo(eventName, x, y, ctrl, shift):
        # helper function to create a string with the event's info
        # also, prints the string for debug info
        msg = ""
        if ctrl:  msg += "ctrl-"
        if shift: msg += "shift-"
        msg += eventName
        msg += " at " + str((x,y))
        print(msg)
        return msg
    
    def leftMousePressed(event):
        #canvas = event.widget.canvas
        ctrl  = ((event.state & 0x0004) != 0)
        shift = ((event.state & 0x0001) != 0)    
        canvas.data["info"] = eventInfo("leftMousePressed", event.x, event.y, ctrl, shift)
        canvas.data["leftPosn"] = (event.x, event.y)
        redrawAll(canvas, data)
        
    def leftMouseMoved(event):
        #canvas = event.widget.canvas
        ctrl  = ((event.state & 0x0004) != 0)
        shift = ((event.state & 0x0001) != 0)
        canvas.data["info"] = eventInfo("leftMouseMoved", event.x, event.y, ctrl, shift)
        canvas.data["leftPosn"] = (event.x, event.y)
        redrawAll(canvas, data)

    def leftMouseReleased(event):
        #canvas = event.widget.canvas
        ctrl  = ((event.state & 0x0004) != 0)
        shift = ((event.state & 0x0001) != 0)
        canvas.data["info"] = eventInfo("leftMouseReleased", event.x, event.y, ctrl, shift)
        canvas.data["leftPosn"] = (200, 100)
        redrawAll(canvas, data)
    
    def mousePressedWrapper(event, canvas, data):
        mousePressed(event, data)
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
    canvas.data = { }
    # set up events
    #root.bind("<Button-1>", lambda event:
     #                       mousePressedWrapper(event, canvas, data))
    root.bind("<Key>", lambda event:
                            keyPressedWrapper(event, canvas, data))
    root.bind("<Button-1>", leftMousePressed)
    canvas.bind("<B1-Motion>", leftMouseMoved)
    root.bind("<B1-ButtonRelease>", leftMouseReleased)
    redrawAll(canvas, data)
    # and launch the app
    root.mainloop()  # blocks until window is closed
    print("bye!")

run(400, 200)
