# mandala
highLevel

# Basic Animation Framework

from tkinter import *

####################################
def getPieSlice(theta, numSlices):
    dAngle=2*pi/numSlices
    minDist=2*pi+1
    minSlice=-1
    for i in range(numSlices):
        angle=dAngle*i
        distance=angle-theta
        if distance==0:
            return None
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

def mousePressReleased(event, data):
    data.lines.append(data.currLine)
    data.currLine = []

def keyPressed(event, data):
    # use event.char and event.keysym
    pass

def redrawAll(canvas, data):
    # draw in canvas
    if "leftPosn" in canvas.data:
        (x, y)=canvas.data["leftPosn"]
        canvas.create_oval(x-2,y-2,x+2,y+2,fill="blue")

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

run(400, 400)
