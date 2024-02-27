<h1>ExpNo 1 :Developing AI Agent with PEAS Description</h1>
<h3>Name: E. Varsha Sharon</h3>
<h3>Register Number:212222100058 </h3>


<h3>AIM:</h3>
<br>
<p>To find the PEAS description for the given AI problem and develop an AI agent.</p>
<br>
<h3>Theory</h3>
<h3>Elevator (Lift) Agent:</h3>
<p>This agent can effectively manage the lift's operation, optimizing performance while ensuring safety and efficiency for passengers. It can make decisions such as which floor to travel to next, when to open or close the doors, and if the number of people entering the lift exceeds the maximum capacity limit of 15, the AI agent will trigger a warning message to ensure safety and adherence to regulations. This warning will alert passengers to reduce the number of occupants inside the lift to a safe level.</p>
<hr>
<h3>PEAS DESCRIPTION:</h3>
<table>
  <tr>
    <td><strong>Agent Type</strong></td>
    <td><strong>Performance</strong></td>
     <td><strong>Environment</strong></td>
    <td><strong>Actuators</strong></td>
    <td><strong>Sensors</strong></td>
  </tr>
    <tr>
    <td><strong>Lift Safety Agent</strong></td>
    <td><strong>Movement between floors and enabling safety measure</strong></td>
     <td><strong>Floors , People, Offices & Buildings</strong></td>
    <td><strong>Lift movement controls (Up / Down) , Door operations (Open/ Close) , Motorst</strong></td>
    <td><strong>Floor sensors , Door status sensors </strong></td>
  </tr>
</table>
<hr>
<H3>DESIGN STEPS</H3>
<h3>STEP 1:Identifying the input:</h3>
<p>The inputs for the Lift Safety Agent: Total number of people, Location.</p>
<h3>STEP 2:Identifying the output:</h3>
<p>Alerting with warning message.</p>
<h3>STEP 3:Developing the PEAS description:</h3>
<p>PEAS description is developed by the performance, environment, actuators, and sensors in an agent.</p>
<h3>STEP 4:Implementing the AI agent:</h3>
<p>Ensuring passenger safety by moving between floors A and B, opening/closing doors based on their status, and waiting when passengers enter or exit, all while adhering to safety regulations.</p>
<h3>STEP 5:</h3>
<p>For each warning the performance is incremented, for each movement with exceeding limit of more than 15 people the performance is decremented.</p>

## PROGRAM:
```
import random
import time

class Thing: 
    """
    This represents any physical object that can appear in an Environment. """

    def is_alive(self):
        """Things that are 'alive' should return true."""
        return hasattr(self, "alive") and self.alive

    def show_state(self):
        """Display the agent's internal state. Subclasses should override."""
        print("I don't know how to show_state.")

class Agent(Thing):
    
    """
        An Agent is a subclass of Thing """

    def __init__(self, program=None):
        self.alive = True
        self.performance = 0 
        self.program = program

    def can_grab(self, thing):
        """Return True if this agent can grab this thing. Override for appropriate subclasses of Agent and Thing.""" 
        return False

def TableDrivenAgentProgram(table): 
    """
    [Figure 2.7]
    This agent selects an action based on the percept sequence. It is practical only for tiny domains.
    To customize it, provide as table a dictionary of all
    {percept_sequence:action} pairs. """
    percepts = []
    
    def program(percept):
        action = None
        percepts.append(percept)
        action = table.get(tuple(percepts))
        return action 
    return program

floor_A, floor_B = (0,0), (1,0) # The two locations for the Lift to move and receive peoples.

def TableDrivenDoctorAgent():
    """
    Tabular approach towards Lift function. 
    """
        
    table = {
    ((floor_A, "closed"),): "up",
    ((floor_A, "opened"),): "wait",
    ((floor_B, "closed"),): "down",
    ((floor_B, "opened"),): "wait",
    ((floor_A, "closed"), (floor_A, "opened")): "up",
    ((floor_A, "clossed"), (floor_B, "opened")): "wait",
    ((floor_B, "closed"), (floor_A, "opened")): "wait",
    ((floor_B, "opened"), (floor_B, "closed")): "down",
    ((floor_A, "opened"), (floor_A, "closed"), (floor_B, "opened")): "wait",
    ((floor_B, "opened"), (floor_B, "closed"), (floor_A, "opened")): "wait",
    }
    return Agent(TableDrivenAgentProgram(table))

TableDrivenDoctorAgent()

class Environment:
    """Abstract class representing an Environment. 'Real' Environment classes inherit from this. Your Environment will typically need to implement:
    percept:	Define the percept that an agent sees. execute_action:	Define the effects of executing an action.
    Also update the agent.performance slot.
    The environment keeps a list of .things and .agents (which is a subset of .things). Each agent has a .performance slot, initialized to 0.
    Each thing has a .location slot, even though some environments may not need this."""

    def __init__(self):
        self.things = [] 
        self.agents = []
        #room_A, room_B = (0,0), (1,0) # The two locations for the Doctor to treat

    def percept(self, agent):
        """Return the percept that the agent sees at this point. (Implement this.)"""
        raise NotImplementedError

    def execute_action(self, agent, action):
        """Change the world to reflect this action. (Implement this.)""" 
        raise NotImplementedError

    def default_location(self, thing):
        """Default location to place a new thing with unspecified location."""
        return None

    def is_done(self):
        """By default, we're done when we can't find a live agent.""" 
        return not any(agent.is_alive() for agent in self.agents)

    def step(self):
        """Run the environment for one time step. If the
        actions and exogenous changes are independent, this method will do. If there are interactions between them, you'll need to override this method."""
        if not self.is_done(): 
            actions = []
            for agent in self.agents:
                if agent.alive:
                    actions.append(agent.program(self.percept(agent))) 
                else:
                    actions.append("")
            for (agent, action) in zip(self.agents, actions): 
                self.execute_action(agent, action)

    def run(self, steps=1000):
        """Run the Environment for given number of time steps."""
        for step in range(steps):
            if self.is_done():
                return 
            self.step()

    def add_thing(self, thing, location=None):
        """Add a thing to the environment, setting its location. For convenience, if thing is an agent program we make a new agent for it. (Shouldn't need to override this.)"""
        if not isinstance(thing, Thing):
            thing = Agent(thing)
        if thing in self.things:
            print("Can't add the same thing twice") 
        else:
            thing.location = (location if location is not None else self.default_location(thing))
            self.things.append(thing) 
            if isinstance(thing, Agent):
                thing.performance = 0 
                self.agents.append(thing)

    def delete_thing(self, thing):
        """Remove a thing from the environment."""
        try:
            
            self.things.remove(thing) 
        except ValueError as e:
            print(e)
            print(" in Environment delete_thing")
            print(" Thing to be removed: {} at {}".format(thing, thing.location))
            print(" from list: {}".format([(thing, thing.location) for thing in self.things]))
        if thing in self.agents: 
            self.agents.remove(thing)

class TrivialDoctorEnvironment(Environment):
    """This environment has two locations, A and B. Each can be unhealthy or healthy. The agent perceives its location and the location's status. This serves as an example of how to implement a simple Environment."""

    def __init__(self):
        super().__init__()
        #room_A, room_B = (0,0), (1,0) # The two locations for the Doctor to treat
        self.status = {floor_A: random.choice(["closed", "opened"]), floor_B: random.choice(["closed", "opened"]),}

    def thing_classes(self):
        return [TableDrivenDocterAgent]

    def percept(self, agent):
        """Returns the agent's location, and the location status (opened/closed)."""
        return agent.location, self.status[agent.location]

    def execute_action(self, agent, action):
        """Change agent's location and/or location's status; track performance. Score 10 for each treatment; -1 for each move."""
        if action == "up":
            agent.location = floor_B
            agent.performance -= 1
        elif action == "down":
            agent.location = floor_A
            agent.performance -= 1
        elif action == "wait":
            fl=float(input("Enter no. of people: ")) 
            if fl>=15:
                self.status[agent.location] == "opened"
                print("""Warning: Maximum Capacity Exceeded. For safety reasons, please reduce the number of passengers in the lift before proceeding.""")
                agent.performance += 10
            else:
                self.status[agent.location] = "closed" 
            self.status[agent.location] = "closed"


    def default_location(self, thing):
           
        return random.choice([floor_A, floor_B])

if   __name__ == "__main__":
    
    agent = TableDrivenDoctorAgent() 
    environment = TrivialDoctorEnvironment() 
    #print(environment)
    environment.add_thing(agent)
    print("\tLift before warning message")
    print(environment.status)
    print("AgentLocation : {0}".format(agent.location)) 
    print("Performance : {0}".format(agent.performance))
    time.sleep(3)
    for i in range(2):
        environment.run(steps=1)
        print("\n\tLift after warning message") 
        print(environment.status)
        print("AgentLocation : {0}".format(agent.location)) 
        print("Performance : {0}".format(agent.performance)) 
        time.sleep(3)
```

## OUTPUT:
![image](https://github.com/varshasharon/19AI405ExpNo1/assets/98278161/7dfdffb2-073c-412d-951d-b0b14116a7e9)

![image](https://github.com/varshasharon/19AI405ExpNo1/assets/98278161/972446e2-efc1-43ba-be34-2bd6697ddae8)

## RESULT:
Thus , Developing an AI agent with PEAS is implemented successfully using python programming. 

