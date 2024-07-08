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


## A full example: restaurant model
Imagine you want to open a restaurant in a popular area of San Francisco. Deciding the number of tables and kitchen capacity is extremely important to ensure the maximum number of customers are served while minimizing initial investment and running costs. A discrete-event model can help in this investment decision by simulating the level of table occupancy, customer waiting times, and the number of customers leaving the queue due to excessive waiting times.

```python
# Functions
def source(env, number, interval, tables):
    """Source generates customers randomly"""
    for i in range(number):
        c = customer(env, f"Customer {i:02d}", tables)
        env.process(c)
        t = random.expovariate(1.0 / interval)
        yield env.timeout(t)


def customer(env, name, tables):
    """
    Generator that simulates table requests and customer decisions to
    wait or leave based on waiting times.
    """
    global customers_served, customers_quiting_waiting
    arrive = env.now

    # Request a table
    with tables.request() as req:
        # Estimate the patience of the customer in minutes
        patience = random.uniform(MIN_PATIENCE, MAX_PATIENCE)

        # Wait until a table is free or the customer runs out of patience
        results = yield req | env.timeout(patience)
        wait = env.now - arrive

    # If customer is served
    if req in results:

      print(f"{env.now:7.4f} {name} > Waited {wait:6.3f} minutes for a table!")
      time_at_tables = random.uniform(MIN_SEATING_TIME, MAX_SEATING_TIME)

      # Yield the time the table is occupied by the customer
      yield env.timeout(time_at_tables)
      print(f"{env.now:7.4f} {name} > Finished meal :)")
      costumers_served += 1

    # If customer leaves the restaurant
    else:
      print(f"{env.now:7.4f} {name} > Gave up waiting and left after waiting {wait:7.4f} minutes :(")
      customers_quiting_waiting += 1

# Run
INTERVAL_CUSTOMERS = 10

# Assign the appropriate values to the model parameters
MIN_PATIENCE = 1
MAX_PATIENCE = 10
MIN_SEATING_TIME = 40
MAX_SEATING_TIME = 90

customers_served = 0
customers_quiting_waiting = 0

env = simpy.Environment()

# Create the SimPy resource
tables = simpy.Resource(env, capacity=2)
env.process(source(env, NEW_CUSTOMERS, INTERVAL_CUSTOMERS, tables))

# Run the model
env.run(until=240)
print(f"Total number of tables served: {customers_served:02d}")
print(f"Total number of customers quiting waiting: {customers_quiting_waiting:02d}")
```

## Building a model: end to end

- Understand the dynamic system
  - What are the aspects that modulate its behavior?
  - What processes/events need to be included?

- Objectives of the model
  - often used to optimize systems

- What are the aspects you want to optimize?
  - more customers
  - faster porcessing times
  - optimize resource allocation
 
- State-variables to monitor these objectives


## MonteCarlo sampling for discrete-event models
When we have a series of sequential processes, different scenarios or *trajectories* arise. A montecarlo sampling analysis can be used to explore this scenarios.

Imagine we have the following manufacturing processes:
```python
porcesses = [{'Name': 'Unloading', 'Average_Duration': 15, 'Standard_Deviation': 4},
 {'Name': 'Cutting', 'Average_Duration': 25, 'Standard_Deviation': 5},
 {'Name': 'Polishing', 'Average_Duration': 10, 'Standard_Deviation': 2},
 {'Name': 'Adding Component 1',
  'Average_Duration': 5,
  'Standard_Deviation': 1},
 {'Name': 'Adding Component 2',
  'Average_Duration': 7,
  'Standard_Deviation': 2},
 {'Name': 'Adding Component 3',
  'Average_Duration': 15,
  'Standard_Deviation': 5},
 {'Name': 'Final assembly', 'Average_Duration': 35, 'Standard_Deviation': 10},
 {'Name': 'Quality verification',
  'Average_Duration': 10,
  'Standard_Deviation': 1},
 {'Name': 'Packaging', 'Average_Duration': 2, 'Standard_Deviation': 0.2}]
```

The Monte-Carlo sampling loop will produce a series of possible process trajectories. You can run it as follows:

```python
n_trajectories = 100

# Run a Monte-Carlo for-loop for n_trajectories samples
for t in range(n_trajectories):

    for p in range(len(processes)):
        proc_p = processes[p]

        # Random gauss method to pseudo-randomly estimate process duration
        process_duration = random.gauss(
            proc_p["Average_Duration"],
            proc_p["Standard_Deviation"]
        )
        time_record[p + 1] = time_record[p] + process_duration

    df_disc = pd.DataFrame({cNam[0]: process_line_space, cNam[1]: time_record})
    fig = sns.lineplot(data=df_disc, x=cNam[0], y=cNam[1], marker="o")  # Step_10
    fig.set(xlim=(0, len(processes) + 1))
    plt.plot()
plt.grid()
plt.show()
```
<img width="749" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/f0b5af17-e10d-41b6-ab59-a335b29b5807">

We can replicate the process using simpy:
```python
def manufacturing_process(env):
    global time_record
    for p in range(len(processes)):
        proc_p = processes[p]
        process_duration = random.gauss(proc_p["Average_Duration"], proc_p["Standard_Deviation"])

        # Clock-in and yield the process_duration
        yield env.timeout(process_duration)

        # Save the current time in time_record
        time_record[p + 1] = env.now

def run_monte_carlo(n_trajectories):

    # Run a for-loop for n_trajectories samples with dummy variable t
    for t in range(n_trajectories):

        # Create the SimPy environment, add processes and run the model
        env = simpy.Environment()
        env.process(manufacturing_process(env))
        env.run()
        
        plot_results()
```

## Idea to structure problems

Imagine we have three processes. We can model each separately, where each returns the times simulated.

```python
def all_processes(env, inputs,record_processes):
    while True:

        time_requests, time_packaging, time_shipping = manage_request(inputs), packaging(inputs), shipment_delivery(inputs)
        yield env.timeout(time_requests)
        yield env.timeout(time_packaging)
        yield env.timeout(time_shipping)

        record_processes['Time Manage Request'].append(time_requests)
        record_processes['Time Packaging'].append(time_packaging)
        record_processes['Time Shipping'].append(time_shipping)

    return record_processes
```

One being, for example:
```python
def manage_request(inputs):
    process_time = rd.gauss(inputs['Managing request of item'][0],
                            inputs['Managing request of item'][1])
    return process_time
```

And run all:
```python
env = simpy.Environment()
env.process(all_processes(env, inputs, record_processes))

# Run the SimPy model
env.run(until=5*365)

record_processes_list = [record_processes['Time Manage Request'],
            			 record_processes['Time Packaging'],
            			 record_processes['Time Shipping']]

# Create a histogram with 50 bins
plt.hist(record_processes_list, bins=50, label=['Request', 'Packaging', 'Shipments'])
plt.legend(loc='upper right')
plt.xlabel('Duration (days)')
plt.ylabel('Number of occurrences')
plt.show()
```

## Clustering

We can cluster trajectories and understand better the causes that lead to each, and inform buisness decisions.

<img width="1486" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/146f26f4-f072-478e-9e17-e8489d0bed3c">


<img width="1482" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/05801a78-2022-4067-86c8-a60d6248bb33">

<img width="1435" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/295dc5f3-0791-448c-a613-7e4a04663942">

```python
def plot_results(objective_func):
    # Score
    fig, axes = plt.subplots(1, len(processes), sharey=True, figsize=(10, 8))
    for p in range(len(processes)):
        sns.scatterplot(ax=axes[p], x=process_duration_all[:, p], y=objective_func, c=objective_func, cmap="turbo_r")
        axes[p].set_title(processes[p]["Name"], rotation = 20, horizontalalignment='left')
        axes[p].set_xlabel("Duration [min]", rotation = -10)
        axes[p].grid()
    axes[0].set_ylabel("Objective function score")
    plt.show()

    # Rank
    index_sort = np.argsort(objective_func)
    fig, axes = plt.subplots(1, len(processes), sharey=True, figsize=(10, 8))
    for p in range(len(processes)):
        sns.lineplot(ax=axes[p], x=np.linspace(0, NUM_SIMULATIONS, NUM_SIMULATIONS),
                     y=process_duration_all[index_sort, p], color="orchid")
        axes[p].set_title(processes[p]["Name"], rotation = 20, horizontalalignment='left')
        axes[p].set_xlabel("Score-ranked scenarios", rotation = -10)
        axes[p].grid()
    axes[0].set_ylabel("Duration [min]")
    plt.show()
```


























