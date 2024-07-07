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
    
**simpy.Container()**

Containers have the following methods: level, put(), get(), capacity. Let´s see them in action in an example. Here, `gas_station_pumps` is the pump server and `gas_station_tank` is the station tank itself. A car arrives at the gas station for refueling. It requests one of the gas station's fuel pumps and tries to get the desired amount of gas from it. If the station's reservoir is depleted, the car has to wait for the tank truck to arrive.


```python

def car_generator(env, gas_station_pumps, gas_station_tank):
    """Generate new cars that arrive at the gas station.
    It ends by triggering a process for each `car` object.
    """
    for i in itertools.count():
        yield env.timeout(random.randint(*T_INTER))
        env.process(car('Car %d' % i, env, gas_station_pumps, gas_station_tank))


def car(name, env, gas_station_pumps, gas_station_tank):
    """A car arrives at the gas station for refueling.

    It requests one of the gas station's fuel pumps and tries to get the
    desired amount of gas from it. If the station's reservoir is
    depleted, the car has to wait for the tank truck to arrive.

    """
    fuel_tank_level = random.randint(*FUEL_TANK_LEVEL)
    print('%s arriving at gas station at %.1f' % (name, env.now))
    with gas_station_pumps.request() as req:
        start = env.now
        # Request one of the gas pumps
        yield req

        # Get the required amount of fuel
        liters_required = FUEL_TANK_SIZE - fuel_tank_level

        # Remove liters_required from the tank
        yield gas_station_tank.get(liters_required)

        # The "actual" refueling process takes some time
        yield env.timeout(liters_required / REFUELING_SPEED)

        print('%s finished refueling in %.1f seconds.' % (name, env.now - start))


# Call tank truck to fill in station
def tank_truck(env, gas_station_tank):   
  yield env.timeout(TANK_TRUCK_TIME)
  print(f"Tank truck arriving at time {env.now}")
  vol_to_refill = gas_station_tank.capacity - gas_station_tank.level
  print(f"Tank truck refuelling {amount} liters")

  # Add fuel to the tank
  yield gas_station_tank.put(vol_to_refill)


# Pumps in action
def gas_station_pumps_control(env, gas_station_tank):
  while True:
    # Check if the level of the tank is below the threshold
    if gas_station_tank.level / gas_station_tank.capacity * 100 < THRESHOLD_PERCENT:

        print(f"Calling tank truck at {env.now}")
        yield env.process(tank_truck(env, gas_station_tank))

    yield env.timeout(10)


env = simpy.Environment()

# Create the gas station resource
gas_station_pumps = simpy.Resource(env, capacity=4)

# Create the gas tank container
gas_station_tank = simpy.Container(env, GAS_STATION_TANK_SIZE, init=GAS_STATION_TANK_SIZE)

# Add processes to the SimPy environment
env.process(gas_station_pumps_control(env, gas_station_tank))
env.process(car_generator(env, gas_station_pumps, gas_station_tank))

env.run(until=SIM_TIME)
```

Then, to inform business decisions we can try increasing the number of pumps until they are not the limiting factor in the system.

```python
random.seed(42)
env = simpy.Environment()
gas_station_pumps = simpy.Resource(env, capacity=6)
gas_station_tank = simpy.Container(env, GAS_STATION_TANK_SIZE, init=GAS_STATION_TANK_SIZE)
env.process(gas_station_pumps_control(env, gas_station_tank))
env.process(car_generator(env, gas_station_pumps, gas_station_tank))
env.run(until=SIM_TIME)
```

## Simpy: managing scheduling of events
The occurrence of events in real-world problems can be affected by several situations. For example:
- conditional events
- interruption of events
- waiting processes
This events can be translated to Simpy discrete events.

### Conditional events
Only triggered if a condition is met. Example: customers arrive at a clothes shop and can buy an item only if it has stock available.

```python
def customer_arrival(env, shop):
    while True:
        yield env.timeout(search_time)

    # Only buy item if available
    if shop.available[item]:
        env.process(shop_buyer(env, item, num_items, shop))
```

### Interruptions
Higher priority events taking place, voluntary interruption or forced interruption. Example: machine breaks periodically, interrupting the production line.

```python
def break_machine(env):
    # break machine every now and then
    while True:
        # time between machine fails
        yield env.timeout(fail_time())

        # machine fail: interrupt process
        simpy_process_1.interrupt()
```

### Waiting processes
Some processes might need others to finish before starting its own process. This happens when resources are limited. 

```python
with gas_station.request():
    # request one of the gas pumps
    yield request

    # once pump is available: pump fuel
    liters_request = 10
    yield fuel_reserve.get(liters_request)

    # the actual refueling takes time
    refuel_duration = liters_request / refuel_speed

    yield env.timeout(refuel_duration)
```

### Concatenate using bitwise operators
Example with movie goers: wait in queue or until tickets are sold out.

```python
with counter.request() as my_turn:
    # wait until turn or movie is sold out
    result = yield my_turn | counter.sold_out[movie]

    # if tickets available
    if my_turn in result:
        theater.num_moviegoers[movie] += 1
        return

    # if tickets sold out
    if my_turn not in result:
        theater.num_renegers[movie] += 1
        return
```




























