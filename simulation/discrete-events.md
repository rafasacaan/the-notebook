# Dynamic systems and discrete events simulations

Three main components:
1. Input data
2. Model parameters
3. Model engine (run all processes)

> Models are only useful if the are informative for decision making.
> Visualization is key!


### Continuous-event models
- Occur in fixed amounts of time (for loop)
- Time is a dummy variable


### Discrete-event models
Components are:
- **state-variables**
    - variables that characterize the output and performance of the system to be studied
- **clock**
- **event list (or process list)**
- **ending condition**


## Simpy
Discrete-event processed based simulation python framework.

1. Create a simpy environment
```python
env = simpy.Environment()
```
2. Generator function of a simpy env
```python
env.process()
```
3. Run the model
```python
env.run()
```
4. Let time pass
```python
env.timeout()
```
5. Get current simulation time
```python
env.now
```

### Simpy example: traffic lights
In this example, we have two traffic lights that operate with different timings. Once we pass both processes and run the simulation, 
both processes will run from time zero in parallel. A unique timeline with events will be output.

```python
import simpy

def traffic_light(env, name, timestep):
  while True:
    yield env.timeout(timestep)
    print(f"Time {env.now:02d} sec | Traffic light at {name} | Red light!")
    yield env.timeout(timestep)
    print(f"Time {env.now:02d} sec | Traffic light at {name} | Yellow light!")
    yield env.timeout(timestep)
    print(f"Time {env.now:02d} sec | Traffic light at {name} | Green light!")

env = simpy.Environment()

env.process(traffic_light(env, "Leslie St.", 15))
env.process(traffic_light(env, "Arlington Av.", 30))

env.run(intil=90)
```

### Simpy example: car wash

```python
import simpy

def car_wash(env):
    car_wash_num = 0
    
    while True:
      car_wash_num += 1

      # Get the current simulation time and add process time
      print(f'Time {env.now:02d} min | Car Wash # {car_wash_num:02d}')
      yield env.timeout(5)

# Create SimPy Environment and add process generator
env = simpy.Environment()
env.process(car_wash(env))

# Run model
env.run(until=8*60)
```

## Simpy type of resources

Three types of resources:
- **simpy.Resource()**
  - share resources between processes. ex: tables in a restaurant, servers in a supermarket
- **simpy.Container()**
  - resource for sharing homogenoeous matter between processes. ex: water (continuous), apples (discrete)
- **simpy.Store()**
  - like Container() but can store unlimited amount of different objects. ex: supermarket 

A variation of the previous resource:
- **simpy.PreemptiveResource()**
  - some ersources may have priority over others: ex: repairman has priority to repair broken machines, and second priority to perform machine maintenance.
 
In order to inform business, we can run different scenarios. For a restaurant example, changing the number of available tables (i.e. servers).  
    






