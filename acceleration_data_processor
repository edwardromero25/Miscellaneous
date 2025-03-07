import matplotlib.pyplot as plt
import numpy as np
from datetime import datetime

A = input("File path: ") 
print(' ')

try:
    with open(A, 'r') as file:
        mainarray = file.read().replace("   ", " ").replace('\t', ' ').replace('\n', ' ').replace(',', ' ').split(' ')
except FileNotFoundError:
    print(f"File not found: {A}")
    exit(1)

datetime_str = []
x = []
y = []
z = []
timestamp = []  

for k in range(0, len(mainarray) - 4, 5):  
    datetime_str.append(mainarray[k] + " " + mainarray[k + 1]) 
    x.append(float(mainarray[k + 2]))
    y.append(float(mainarray[k + 3]))
    z.append(float(mainarray[k + 4]))

datetime_obj = [datetime.strptime(dt, '%H:%M:%S %m/%d/%Y') for dt in datetime_str]

time_in_seconds = [(dt - datetime_obj[0]).total_seconds() for dt in datetime_obj]
time_in_hours = [t / 3600 for t in time_in_seconds] 

class PathVisualization:
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

        self.pathCoords = list(zip(self.x, self.y, self.z))
        self.num_points = 1000

    def __createSphere(self):
        golden_r = (np.sqrt(5.0) + 1.0) / 2.0            
        golden_a = (2.0 - golden_r) * (2.0 * np.pi)     

        Xs, Ys, Zs = [], [], []
        
        for i in range(self.num_points):
            ys = 1 - (i / float(self.num_points - 1)) * 2
            radius = np.sqrt(1 - ys * ys)

            theta = golden_a * i

            xs = np.cos(theta) * radius
            zs = np.sin(theta) * radius

            Xs.append(xs)
            Ys.append(ys)
            Zs.append(zs)

        return(Xs, Ys, Zs)

    def __splitSphere(self, sphereCoords):
        octants = {'posI':[], 'posII':[], 'posIII':[], 'posIV':[], 'negI':[], 'negII':[], 'negIII':[], 'negIV':[]}
        
        for row in sphereCoords:
            if (row[2] > 0):
                if (row[1] > 0):
                    if (row[0] > 0):
                        octants['posI'].append(row)
                    else:
                        octants['posII'].append(row)
                elif (row[0] > 0):
                    octants['posIV'].append(row)
                else: 
                    octants['posIII'].append(row)
            else:
                if (row[1] > 0):
                    if (row[0] > 0):
                        octants['negI'].append(row)
                    else:
                        octants['negII'].append(row)
                elif (row[0] > 0):
                    octants['negIV'].append(row)
                else:
                    octants['negIII'].append(row)
        
        return(octants)

    def __getPathOctant(self, pathRow):
        if (pathRow[2] > 0):
            if (pathRow[1] > 0):
                if (pathRow[0] > 0):
                    return 'posI'
                else:
                    return 'posII'
            elif (pathRow[0] > 0):
                return 'posIV'
            else: 
                return 'posIII'
        else:
            if (pathRow[1] > 0):
                if (pathRow[0] > 0):
                    return 'negI'
                else:
                    return 'negII'
            elif (pathRow[0] > 0):
                return 'negIV'
            else:
                return 'negIII'

    def __getDistanceBetween(self, pathTupleCoords, sphereTupleCoords):
        pathX, pathY, pathZ = pathTupleCoords
        sphereX, sphereY, sphereZ = sphereTupleCoords

        diffX = pathX - sphereX
        diffY = pathY - sphereY
        diffZ = pathZ - sphereZ

        sumSquares = diffX ** 2 + diffY ** 2 + diffZ ** 2
        dist = np.sqrt(sumSquares)

        return dist

    def __getDistributionNum(self, sphereCoords):
        octants = self.__splitSphere(sphereCoords)
        
        pathMap = {} 
        repeatTime = -1

        for pathRow in self.pathCoords:
            pathOctant = self.__getPathOctant(pathRow)
            sphereCoordsSplit = octants[pathOctant]
            distDict = {}
            repeatTime += 1

            for sphereRow in sphereCoordsSplit:
                dist = self.__getDistanceBetween(pathRow, sphereRow) 
                distDict[sphereRow] = dist 
                    
            rankedDist = sorted(distDict.items(), key=lambda x:x[1]) 
            segmentVertices = (rankedDist[0][0], rankedDist[1][0], rankedDist[2][0]) 
            
            pathMap[segmentVertices] = pathMap.get(segmentVertices, []) + [repeatTime] 
        
        return(len(pathMap))

    def getDistribution(self):
        Xsphere, Ysphere, Zsphere = self.__createSphere()
        sphereCoords = list(zip(Xsphere, Ysphere, Zsphere))
        score = self.__getDistributionNum(sphereCoords)
        return score

class PathFigure:
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

    def createPathFig(self, mode='show', title=True):
        plt.rcParams['font.family'] = 'Calibri'
        fig = plt.figure(figsize=plt.figaspect(0.85))

        if title:
            fig.suptitle("Acceleration Vector Path")

        ax = fig.add_subplot(1, 1, 1, projection='3d')
        ax.plot(self.x, self.y, self.z, color='#0032A0', linewidth=1)

        ax.set_xlabel('X')
        ax.set_ylabel('Y')
        ax.set_zlabel('Z')

        ticks = np.arange(-1.0, 1.5, 0.5)
        ax.set_xticks(ticks)
        ax.set_yticks(ticks)
        ax.set_zticks(ticks)

        path_visualization = PathVisualization(self.x, self.y, self.z)
        distribution_score = path_visualization.getDistribution()
        ax.legend([f"Distribution: {distribution_score}"])

        if mode == 'save':
            plt.savefig('pathFig.png')
        elif mode == 'show':
            plt.show()
        else:
            plt.savefig('pathFig.png')
            plt.show()

class AccelerometerDataProcessor:
    def __init__(self, x, y, z, time_in_hours, startAnalysis, endAnalysis):
        self.x = x
        self.y = y
        self.z = z
        self.time_in_hours = time_in_hours
        self.maxSeg = len(x) - 1
        self.startAnalysis = startAnalysis
        self.endAnalysis = endAnalysis

    def _getTimeAvg(self):
        xTimeAvg = np.cumsum(self.x) / np.arange(1, len(self.x) + 1)
        yTimeAvg = np.cumsum(self.y) / np.arange(1, len(self.y) + 1)
        zTimeAvg = np.cumsum(self.z) / np.arange(1, len(self.z) + 1)
        return xTimeAvg, yTimeAvg, zTimeAvg

    def _getMagnitude(self, xTimeAvg, yTimeAvg, zTimeAvg):
        magList = np.sqrt(xTimeAvg**2 + yTimeAvg**2 + zTimeAvg**2)
        return magList

    def _getMagSeg(self, magList):
        startSeg = next(i for i, t in enumerate(self.time_in_hours) if t >= self.startAnalysis)
        endSeg = next(i for i, t in enumerate(self.time_in_hours) if t >= self.endAnalysis)
        avgMagFull = np.mean(magList)
        avgMagAnalysis = np.mean(magList[startSeg:endSeg])
        return avgMagFull, avgMagAnalysis

    def createMagFig(self, mode='show', title=True):
        xTimeAvg, yTimeAvg, zTimeAvg = self._getTimeAvg()
        magList = self._getMagnitude(xTimeAvg, yTimeAvg, zTimeAvg)
        avgMagFull, avgMagAnalysis = self._getMagSeg(magList)

        fig = plt.figure()
        ax = fig.add_subplot(1, 1, 1)
        plt.yscale('log')

        if title:
            fig.suptitle("Magnitude vs. Time")

        ax.plot(self.time_in_hours, magList, color='#0032A0', label="Average Magnitude: " + f"{avgMagFull:.3g}")

        startSeg = next(i for i, t in enumerate(self.time_in_hours) if t >= self.startAnalysis)
        endSeg = next(i for i, t in enumerate(self.time_in_hours) if t >= self.endAnalysis)

        ax.axvline(x=self.startAnalysis, color='#E4002B', linestyle='--')
        ax.axvline(x=self.endAnalysis, color='#E4002B', linestyle='--')

        ax.plot(self.time_in_hours[startSeg:endSeg], magList[startSeg:endSeg], color='#E4002B', label="Average Magnitude (Analysis): " + f"{avgMagAnalysis:.3g}")

        ax.legend()
        ax.set_xlabel('Time (hours)')
        ax.set_ylabel('Magnitude (g)')

        if mode == 'save': 
            plt.savefig('timeMagFig.png')
        elif mode == 'show':
            plt.show()
        else:
            plt.savefig('timeMagFig.png')
            plt.show()

startAnalysis = float(input("Enter the start time for analysis in hours: "))
endAnalysis = float(input("Enter the end time for analysis in hours: "))

processor = AccelerometerDataProcessor(x, y, z, time_in_hours, startAnalysis, endAnalysis)
processor.createMagFig(mode='show')

path_figure = PathFigure(x, y, z)
path_figure.createPathFig(mode='show')
