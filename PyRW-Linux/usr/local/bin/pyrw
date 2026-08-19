#!/usr/bin/env python3
# Made by @Ericwasepic127

import time, tempfile, tkinter as tk, os
from tkinter import simpledialog as sD, messagebox as mB
from threading import Thread  

lbl = None
infos = "Total benchmark size: %s %s | Chunk size: %s %s"
TYPE_KB = 0
TYPE_MB = 1
TYPE_GB = 2
SELECTED = 2
SELECTED2 = 1
SELECTIONS = [ "KiloBytes (KB)", "MegaBytes (MB)", "GigaBytes (GB)"]
NO_VERBOSE = lambda l: None
formatted = """
Results:
- Total size of speed meter used: %s %s
- Chunk size: %s %s
- Reading speed: %s
- Writing speed: %s
"""

def byte(size: int, types=TYPE_MB) -> int:
    """Converts to byte"""
    for _ in "a"*(types+1):
       size *= 1024
    return size

def cbyte(size: int, types=TYPE_MB) -> int:
    """Converts from byte"""
    for _ in "a"*(types + 1):
       size /= 1024
    return size

SIZE = byte(1, types=TYPE_GB)
CHUNK = byte(4)

def meter(size=byte(1, types=TYPE_GB), chunk=byte(4, types=TYPE_MB), verbose=print) -> list:
    """Returns speed of read & writing given size by given byte, with chunking"""
    results = [size]
    sizeW = size
    sizeR = 0
    with tempfile.TemporaryFile(mode="w+b") as tmp:
        start = time.time()
        while round(sizeW) > 0:
            tmp.write(b"\x01"*int(chunk))
            verbose(f"[INFO]: Writing, {sizeW} bytes left")
            sizeW -= int(chunk)
        stop = time.time() - start
        results.append(float(f"{stop:.02f}"))
        tmp.flush()
        os.fsync(tmp.fileno())
        tmp.seek(0)
        start = time.time()
        while sizeR <= size:
            sizeR += int(chunk)
            tmp.read(sizeR)
            verbose(f"[INFO]: Reading, has {sizeR} bytes readed")
        stop = time.time() - start
        results.append(float(f"{stop:.02f}"))
    return results

def speedMbs(times: list):
    size = times[0]
    write = times[1]
    read = times[2]
    final = []
    if write == 0:
        final.append("? MB/s")
    else:
        final.append("{:.2f} MB/s".format((size / write) / 1048576))
    if read == 0:
        final.append("? MB/s")
    else:
        final.append("{:.2f} MB/s".format((size / read) / 1048576))
    return final

def texter(obj, text, to=tk.END, end="\n"):
    obj["state"] = "normal"
    obj.insert(to, text + end)
    obj['state'] = 'disabled'
    obj.see(tk.END)

def run(size, chunk, verbose=print, root=None):
    try:
        res = meter(size, chunk, verbose)
    except Exception as e:
        verbose("[ERROR]: Failed to detect speed: %s" % str(e))
        if root is not None:
            x = tk.Button(root[0], text="EXIT", command=root[1])
            x.pack()
            root[0].bind("<Return>", root[1])
            verbose("[INFO]: Press Exit button or enter to quit")
        return
    verbose("[INFO]: Calculating speed ...")
    fin = speedMbs(res)
    vals = valExtract()
    if vals:
        verbose(formatted % (*vals, fin[1], fin[0]))
    else:
       verbose(formatted % (size, "bytes", chunk, "bytes", fin[1], fin[0]))
    if root is not None:
        x = tk.Button(root[0], text="EXIT", command=root[1])
        x.pack()
        root[0].bind("<Return>", root[1])
        verbose("[INFO]: DONE - press Exit button or enter to quit")
    else:
        verbose("[INFO]: DONE")

def display(size, chunk, rootTK):
    rootTK.withdraw()
    child = tk.Toplevel(rootTK)
    child.geometry("500x500")
    child.title("RESULTS")
    child.resizable(False, False)
    def destroy(xd=None):
        child.destroy()
        rootTK.deiconify() if mB.askyesno("Benchmark again?", "Would you rather benchmark again?")  else None
    tk.Label(child, text="LOGS:").pack()
    txt = tk.Text(child, state="disabled")
    txt.pack(fill=tk.BOTH)
    output = lambda l: texter(txt, l)
    Thread(daemon=True, target=lambda: run(size, chunk, output, (child, destroy))).start()
    child.mainloop()
 
def onSelect(has, btn):
    global SELECTED
    SELECTED = SELECTIONS.index(has)
    updateVals(lbl)

def onSelect2(has, btn):
    global SELECTED2
    SELECTED2 = SELECTIONS.index(has)
    updateVals(lbl)

def changeSize():
    global SIZE
    SIZE = byte(sD.askfloat("New size value", f"Please enter new size value in {SELECTIONS[SELECTED]}") or SIZE, types=SELECTED)
    updateVals(lbl)

def changeSize2():
    global CHUNK
    CHUNK = byte(sD.askfloat("New size value", f"Please enter new size value in {SELECTIONS[SELECTED2]}") or CHUNK, types=SELECTED2)
    updateVals(lbl)

def start(root):
    display(SIZE, CHUNK, root)

def valExtract():
    s = SELECTIONS[SELECTED]
    s2 = SELECTIONS[SELECTED2]
    return (cbyte(SIZE, SELECTED), s, cbyte(CHUNK, SELECTED2), s2)

def updateVals(obj=lbl):
    if obj:
        lbl["text"] = infos % valExtract()

def main():
    global lbl
    root = tk.Tk()
    root.geometry("545x200")
    root.title("Read/Write detector")
    root.resizable(False, False)

    tk.Label(root, text="Welcome to Read and Write speed meter for your drive!").pack()

    lbl = tk.Label(root)
    lbl.pack()
    updateVals(lbl)

    root1 = tk.Frame(root)
    root1.pack(pady=5)
    var = tk.StringVar(root1)
    var.set(SELECTIONS[2])
    btn = tk.Button(root1, text=f"Change how much size will use (more means more accurate)", command=changeSize)
    btn.pack(side=tk.LEFT, padx=5)
    tk.OptionMenu(root1, var, *SELECTIONS, command=lambda x: onSelect(x, btn)).pack(side=tk.LEFT, padx=5)
    
    root2 = tk.Frame(root)
    root2.pack(pady=5)
    var2 = tk.StringVar(root2)
    var2.set(SELECTIONS[1])
    btn2 = tk.Button(root2, text=f"Change how much size to use in per chunk read & write", command=changeSize2)
    btn2.pack(side=tk.LEFT, padx=5)
    tk.OptionMenu(root2, var2, *SELECTIONS, command=lambda x: onSelect2(x, btn2)).pack(side=tk.LEFT, padx=5)

    tk.Button(root, text="RUN TEST", command=lambda: start(root)).pack()
    root.bind("<Return>", lambda x: start(root))
    
    root.mainloop()

if __name__ == '__main__':
    main()
