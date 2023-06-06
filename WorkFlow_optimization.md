```python
pip install gurobipy
```

    Looking in indexes: https://pypi.org/simple, https://us-python.pkg.dev/colab-wheels/public/simple/
    Collecting gurobipy
      Downloading gurobipy-10.0.1-cp39-cp39-manylinux2014_x86_64.whl (12.8 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m12.8/12.8 MB[0m [31m18.5 MB/s[0m eta [36m0:00:00[0m
    [?25hInstalling collected packages: gurobipy
    Successfully installed gurobipy-10.0.1



```python
import sys
import time
import json
import gurobipy as gp
import random
from gurobipy import *
from gurobipy import GRB
```


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive



```python
#import graph

import json
import networkx as nx

with open('/content/drive/MyDrive/Group Project_Big Graph/Code/Final version/graph.json','r') as f:
      data = json.load(f)

G = nx.Graph()
G.add_nodes_from(data['nodes'])
G.add_edges_from(data['edges'])

n = len(G.nodes)
I = G.edges()
```


```python
try :     
    model = gp.Model("assembly_scheduling_problem")

    # Parameters **
    n = 10 # Number of operations n (= number of nodes in graph)
    N = [i+1 for i in range(n)] # set of n operations 

    # Define the O dictionary to denote each operation i
    O = {i+1: f"O[{i+1}]" for i in range(n)}

    k = 10 # example of w work centers
    Y = [i+1 for i in range(k)] # set of w work centers
    #f[Y]= num of machines in workcenter Y
    f = [random.randint(1, 5) for _ in range(len(Y))]
    #define I[Y] : set of operations that requires workcenter Y
    #I_Y= {y: [] for y in Y}
  
    #I = [(1, 2), (1, 3), (2, 4), (2, 5), (3, 4), (3, 5), (4, 7), (5, 6), (5, 7)]

    s = {}
    for s_j, j in I:
        s[j] = s_j
  

    #Variables 

    # Z (ub = infinity)
    Z = model.addVar(0.0,GRB.INFINITY,0.0,vtype=GRB.CONTINUOUS, name="Z")

    # S_j : starting time of operation j
    S = model.addVars(N, vtype=GRB.CONTINUOUS, name="S")
    # ES_j : earliest starting time of operation j 
    ES= model.addVars(N, vtype=GRB.CONTINUOUS, name="ES")

    # F_j Finishing time of operation j 
    F = model.addVars(N, vtype=GRB.CONTINUOUS, name="F")

    # Define the range for random processing times
    min_processing_time = 1
    max_processing_time = 5
    # Generate random processing times for each operation
    t = {}
  
    for j in N:
        t[j] = random.randint(min_processing_time, max_processing_time)

    print(t)
    # h_j index of work center - operation j is designated to 
    # create dictionary to holde the decision var 
    h = model.addVars(N, vtype=GRB.INTEGER, name="h")


    #Binary variable Psi = (V)
    V = model.addVars(N,N, vtype=GRB.BINARY, name="V")
    for i, j in I:
        if j in s and s[j] == i:
            model.addConstr(V[i, j] == 1, name=f"V_constr_{i}_{j}")
        else:
            model.addConstr(V[i, j] == 0, name=f"V_constr_{i}_{j}")

    #Binary variable Phi[j][y]

    phi = model.addVars(N, Y, vtype=GRB.BINARY, name="phi")
    #some_condition=True
    for j in N:
        for y in Y:
                if j in s:
                    model.addConstr(phi[j, y] == 1, name=f"phi_constr_{j}_{w}")
                else:
                    model.addConstr(phi[j, y] == 0, name=f"phi_constr_{j}_{w}")

    #binary variable X[i][j]         
    X = model.addVars(I, vtype=GRB.BINARY, name="X")
    for i, j in I:
        if j in s and s.get(i) == j:
            model.addConstr(X[i, j] == 1, name=f"X_constr_{i}_{j}")
        else:
            model.addConstr(X[i, j] == 0, name=f"X_constr_{i}_{j}")



    # add constraints 
    M = 1000  # Large constant value for constraints
    # Constraints1
    for i, j in I:
            model.addConstr(S[j] >= F[i]) 
           #model.addConstr(S[j] >= F[i] - M * (1 - X[i, j]))

    for j in N:
        model.addConstr(S[j] >= ES[j])  # constraint 2
        model.addConstr(F[j] <= Z)  # constraint 3
        model.addConstr(F[j] == S[j] + t[j])  # constraint 4
    # Constraint 5
    for i in N:
        for j in N:
            if i != j:
                model.addConstr(V[i, j] + V[j, i] == 1, name=f"constr5_{i}_{j}")
    # constraint 6                    
    for i, j in I:
        if i != j:
            for w in range(len(Y)):
                for y in range(1,f[w]+1): 
                    model.addConstr(S[i] - F[j] >= (V[i, j] + phi[j, y] + phi[i, y] - 3) * M)
    # constraint 7
  
    for w in range(len(f)):
        for j in N:
            model.addConstr(sum(phi[j,y] for y in range(1, f[w]+1)) == 1)

    #Objective function: minimize total processing time
    model.setObjective(Z, GRB.MINIMIZE)

    model.optimize()
except Exception as e:
    print("Cannot progress the program")
```

    {1: 3, 2: 5, 3: 1, 4: 3, 5: 2, 6: 2, 7: 1, 8: 2, 9: 2, 10: 5}
    Gurobi Optimizer version 10.0.1 build v10.0.1rc0 (linux64)
    
    CPU model: Intel(R) Xeon(R) CPU @ 2.20GHz, instruction set [SSE2|AVX|AVX2]
    Thread count: 1 physical cores, 2 logical processors, using up to 2 threads
    
    Optimize a model with 617 rows, 250 columns and 2026 nonzeros
    Model fingerprint: 0x7c9deded
    Variable types: 31 continuous, 219 integer (209 binary)
    Coefficient statistics:
      Matrix range     [1e+00, 1e+03]
      Objective range  [1e+00, 1e+00]
      Bounds range     [1e+00, 1e+00]
      RHS range        [1e+00, 3e+03]
    Presolve removed 69 rows and 89 columns
    Presolve time: 0.00s
    
    Explored 0 nodes (0 simplex iterations) in 0.01 seconds (0.00 work units)
    Thread count was 1 (of 2 available processors)
    
    Solution count 0
    
    Model is infeasible
    Best objective -, best bound -, gap -

