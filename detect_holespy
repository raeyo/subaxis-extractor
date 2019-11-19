from __future__ import print_function
import sys
sys.path.append("/usr/lib/freecad/lib/")
import FreeCAD 
import FreeCADGui 
import Part, Mesh
from FreeCAD import Draft
from PyQt4 import QtGui,QtCore
from math import pi
import random
import argparse
import time
import glob
from tqdm import tqdm
import json

def detect_holes(step_file, doc_name):

    # read step file
    shape = Part.Shape()
    shape.read(step_file)

    # initialize FreeCAD
    FreeCADGui.showMainWindow()
    doc = FreeCAD.newDocument(doc_name)
    FreeCAD.setActiveDocument(doc_name)
    FreeCAD.ActiveDocumnet = FreeCAD.getDocument(doc_name)
    FreeCADGui.ActiveDocument

    # load object
    pf = doc.addObject("Part::Feature","MyShape")
    pf.Shape = shape
    FreeCAD.ActiveDocument.recompute()
    FreeCADGui.getDocument(doc_name)

    # eliminate flat faces
    facelist = []
    for f in pf.Shape.Faces:
        if f.ParameterRange[1] > 6.28318 and f.ParameterRange[1] < 6.283186 :
            facelist.append(f)

    # detect holes
    holes = []
    hole_id = 0
    for h in facelist:
        for w in h.Wires:
            for c in w.Edges:
                # find any cylinderical objects
                if (isinstance(c.Curve, Part.Circle)):
                    hole = {}
                    print(c.Curve)
                    hole["hole_id"] = hole_id
                    hole["radius"] = c.Curve.Radius
                    hole["position"] = (c.Curve.Center.x, c.Curve.Center.y, c.Curve.Center.z)
                    hole["x_axis"] = (c.Curve.XAxis.x, c.Curve.XAxis.y, c.Curve.XAxis.z)
                    hole["y_axis"] = (c.Curve.YAxis.x, c.Curve.YAxis.y, c.Curve.YAxis.z)
                    hole["z_axis"] = (c.Curve.Axis.x, c.Curve.Axis.y, c.Curve.Axis.z)

                    holes.append(hole)
                    hole_id += 1

    if args.gui != 0:
        for hole in holes:
            # get z-axis
            for i, normal in enumerate(hole["z_axis"]):
                if int(normal) != 0:
                    z_axis = i
            pos1 = list(hole["position"])
            pos2 =list(hole["position"])
            pos1[z_axis] += 30
            pos2[z_axis] -= 30

            obj = doc.addObject("Part::Polygon", "Polygon")
            obj.Nodes = [tuple(pos1), tuple(pos2)]

        FreeCAD.ActiveDocument.recompute()
        FreeCADGui.getDocument(doc_name)
        FreeCADGui.showMainWindow()

        view = FreeCADGui.ActiveDocument.ActiveView
        view.viewAxometric()
        view.fitAll()

        for i in range(args.gui*20):
            FreeCADGui.updateGui()
            time.sleep(0.05)

    if args.obj:
        obj_path = './obj/' + step_file.split('/')[-1].split('.')[0] + '.obj'
        import Mesh
        Mesh.export([pf], obj_path)
    return holes


if __name__ == "__main__":

    parser = argparse.ArgumentParser()

    parser.add_argument("-i","--input_path", type=str, default='./step/stefan', help="path for the step files")
    parser.add_argument("-o","--output_path", type=str, default='./hole_info', help="path for the output json file")
    parser.add_argument("--gui", type=int, default='0', help="enable gui mode for x seconds")
    parser.add_argument("--obj", action='store_true', help="convert step file to obj and save it")
    parser.add_argument("--doc", type=str, default='doc', help="document name")

    args = parser.parse_args()

    input_paths = glob.glob(args.input_path + '/*')
    input_paths.sort()

    file_id = 0

    for i, input_path in enumerate(tqdm(input_paths)):
        hole_info = {}
        hole_info['file_id'] = file_id
        file_name = input_path.split('/')[-1].split('.')[0]
        hole_info['file_name'] = file_name

        if file_name.find('ea)') == -1:
            hole_info['EA'] = 1
        else:
            hole_info['EA'] = int(file_name[file_name.find('(')+1:file_name.find('ea)')])

        print("detect {} holes in {}".format(hole_info['EA'], hole_info['file_name']), '\n')
        holes = detect_holes(input_path, args.doc + str(i))
        hole_info["hole"] = holes
        file_id += 1

        # save dictionary into json
        with open(args.output_path + '/' + file_name + '.json', 'w') as f:
            json.dump(hole_info, f)
