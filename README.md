# Developing AI Agent with PEAS Description

## AIM

To find the PEAS description for the given AI problem and develop an AI agent.

## THEORY
Lift is a agent here.Two locations namely loc A (groundf floor)and loc B(first floor).If the lift door open in loc A ,the passengers will go up.If the lift door open in loc B ,the passengers will go down.If lift door close at loc A,the passenger will wait.If lift door close at loc B,the passenger will wait.



## PEAS DESCRIPTION
| Agent type    | performance  measurement    |environment  |   Actuators         |  sensors                       | 
|-------------  | --------------------------- | ----------- |-------------------- | ------------------------------ | 
| Lift          | lifting passengers          | loc A,loc B | Buttons, door       |    touch sensor                |


## DESIGN STEPS
### STEP 1:
Identifying the input:

### STEP 2:
Identifying the output:

### STEP 3:
Developing the PEAS description:
### STEP 4:
Implementing the AI agent

### STEP 5:
Measure the performance parameters

## PROGRAM
```python
import random

class Thing:
    """
        This represents any physical object that can appear in an Environ
        ment.
    """

    def is_alive(self):
        """Things that are 'alive' should return true."""
        return hasattr(self, 'alive') and self.alive

    def show_state(self):
        """Display the agent's internal state. Subclasses should override."""
        print("I don't know how to show_state.")
class Agent(Thing):
    """
        An Agent is a subclass of Thing
    """

    def __init__(self, program=None):
        self.alive = True
        self.performance = 0
        self.program = program

    def can_grab(self, thing):
        """Return True if this agent can grab this thing.
        Override for appropriate subclasses of Agent and Thing."""
        return True
def TableDrivenAgentProgram(table):
    """
    [Figure 2.7]
    This agent selects an action based on the percept sequence.
    It is practical only for tiny domains.
    To customize it, provide as table a dictionary of all
    {percept_sequence:action} pairs.
    """
    percepts = []

    def program(percept):
        percepts.append(percept)
        action = table.get(tuple(percepts))
        return action

    return program

loc_A, loc_B = (0, 0), (0, 1)  # The two locations for the lift
def TableDrivenVacuumAgent():
    """
    Tabular approach towards vacuum world
    """
    table = {((loc_A, 'open'),): 'up',
             ((loc_A, 'close'),): 'wait',
             ((loc_B, 'open'),): 'down',
             ((loc_B, 'close'),): 'wait',
             ((loc_A, 'close'), (loc_A, 'open')): 'up',
             ((loc_A, 'open'), (loc_B, 'close')): 'wait',
             ((loc_B, 'open'), (loc_A, 'close')): 'wait',
             ((loc_B, 'close'), (loc_B, 'open')): 'down',
             ((loc_A, 'close'), (loc_A, 'open'), (loc_B, 'close')): 'wait',
             ((loc_B, 'close'), (loc_B, 'open'), (loc_A, 'close')): 'wait',
             ((loc_A, 'open'),  (loc_B, 'open'), (loc_A, 'open')): 'up',
             ((loc_B, 'open'), (loc_A, 'close'), (loc_B, 'open')): 'down',
             ((loc_A, 'open'), (loc_A, 'close'), (loc_B, 'open'),(loc_A, 'open')): 'wait',
             ((loc_B, 'open'), (loc_B, 'open'), (loc_A, 'open'),(loc_B, 'open')): 'wait',
             ((loc_A, 'close'),  (loc_B, 'close'), (loc_A, 'close'),(loc_A, 'close')): 'up',
             ((loc_B, 'close'), (loc_A, 'open'), (loc_B, 'close'),(loc_B, 'close')): 'down',
             ((loc_A, 'open'), (loc_A, 'close'), (loc_B, 'open'),(loc_A, 'open')): 'wait',((loc_A, 'close'),): 'wait',
             ((loc_B, 'open'), (loc_B, 'open'), (loc_A, 'open'),(loc_B, 'open')): 'wait',((loc_A, 'open'),): 'up',
             ((loc_A, 'close'),  (loc_B, 'close'), (loc_A, 'close'),(loc_A, 'close')): 'up',((loc_B, 'close'),): 'wait',
             ((loc_B, 'close'), (loc_A, 'open'), (loc_B, 'close'),(loc_B, 'close')): 'down',((loc_B, 'open'),): 'down' }
    return Agent(TableDrivenAgentProgram(table))
class Environment:
    """Abstract class representing an Environment. 'Real' Environment classes
    inherit from this. Your Environment will typically need to implement:
        percept:           Define the percept that an agent sees.
        execute_action:    Define the effects of executing an action.
                           Also update the agent.performance slot.
    The environment keeps a list of .things and .agents (which is a subset
    of .things). Each agent has a .performance slot, initialized to 0.
    Each thing has a .location slot, even though some environments may not
    need this."""

    def __init__(self):
        self.things = []
        self.agents = []

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
        actions and exogenous changes are independent, this method will
        do. If there are interactions between them, you'll need to
        override this method."""
        if not self.is_done():
            actions = []
            for agent in self.agents:
                if agent.alive:
                    actions.append(agent.program(self.percept(agent)))
                else:
                    actions.append("")
            for (agent, action) in zip(self.agents, actions):
                self.execute_action(agent, action)

    def run(self, steps=3000):
        """Run the Environment for given number of time steps."""
        for step in range(steps):
            if self.is_done():
                return
            self.step()

    def add_thing(self, thing, location=None):
        """Add a thing to the environment, setting its location. For
        convenience, if thing is an agent program we make a new agent
        for it. (Shouldn't need to override this.)"""
        if not isinstance(thing, Thing):
            thing = Agent(thing)
        if thing in self.things:
            print("Can't add the same thing twice")
        else:
            thing.location = location if location is not None else self.default_location(thing)
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
            print("  in Environment delete_thing")
            print("  Thing to be removed: {} at {}".format(thing, thing.location))
            print("  from list: {}".format([(thing, thing.location) for thing in self.things]))
        if thing in self.agents:
            self.agents.remove(thing)
class TrivialVacuumEnvironment(Environment):
    """This environment has two locations, A and B. Each can be open
    or Close. The agent perceives its location and the location's
    status. This serves as an example of how to implement a simple
    Environment."""

    def __init__(self):
        super().__init__()
        self.status = {loc_A: random.choice(['open', 'close']),
                       loc_B: random.choice(['open', 'close'])}

    def thing_classes(self):
        return [ TableDrivenVacuumAgent]

    def percept(self, agent):
        """Returns the agent's location, and the location status (/open)."""
        return agent.location, self.status[agent.location]

    def execute_action(self, agent, action):
        """Change agent's location and/or location's status; track performance.
        Score 10 for each open opened; -1 for each move."""

       
        if action=='up':
            agent.location = loc_A
            agent.performance -=1
        elif action=='down':
            agent.location = loc_B
            agent.performance -=1
        elif action=='wait':
            if self.status[agent.location]=='close':
                agent.performance+=10
            self.status[agent.location]='open'
        
          
       

    def default_location(self, thing):
        """Agents start in either location at random."""
        return random.choice([loc_A, loc_B])
    
  if __name__ == "__main__":
    agent = TableDrivenVacuumAgent()
    environment = TrivialVacuumEnvironment()
    environment.add_thing(agent)
    print("before execution:")
    print(environment.status)
    print("agent location:{0}".format(agent.location))
    print("performance:{0}".format(agent.performance))
    environment.run()
    print("after execution")
    print(environment.status)
    print("agent location:{0}".format(agent.location))
    print("performance:{0}".format(agent.performance))
   
```

## OUTPUT
![Screenshot (40)](https://user-images.githubusercontent.com/75243072/162258417-f79de2f7-168d-496d-8934-e0cfd33fe4e9.png)



## RESULT
Thus, an AI agent was developed and PEAS description is given.


