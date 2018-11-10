# mandala
highLevel

# Basic Animation Framework

from tkinter import *

####################################
def getPieSlice(theta, numSlices):
    dAngle=360/numSlices
    minDist=361
    minSlice=-1
    for i in range(numSlices):
        angle=dAngle*i
        distance=angle-theta
        if distance<minDist:
            minDist=distance
            minSlice=i
    return minSlice

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
    if "leftPosn" in canvas.data:
        (x, y)=canvas.data["leftPosn"]
        canvas.create_oval(x-2,y-2,x+2,y+2,fill="blue")

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
