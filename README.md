# MIT4 labor

## 2. feladat forráskódja:

```java
package rescueagents;

import rescueframework.AbstractRobotControl;
import world.Cell;
import world.Path;
import world.Robot;
import world.RobotPercepcion;

public class RobotControl extends AbstractRobotControl{

    public RobotControl(Robot robot, RobotPercepcion percepcion) {
        super(robot, percepcion);
    }
    
    public Integer step() {      
    	Cell location = robot.getLocation();
    	Path exitPath = percepcion.getShortestExitPath(location);
    	Path unkPath  = percepcion.getShortestUnknownPath(location);
    	Path injPath  = percepcion.getShortestInjuredPath(location);
        if(robot.hasInjured()) {
        	return exitPath.getFirstCell().directionFrom(location);
        }else if(injPath != null) {
        	return injPath.getFirstCell().directionFrom(location);
        }else {
        	return unkPath.getFirstCell().directionFrom(robot.getLocation());
        }
    }
}

```

## 3. feladat forráskódja:

```java
package rescueagents;

import java.util.ArrayList;

import rescueframework.AbstractRobotControl;
import world.AStarSearch;
import world.Cell;
import world.Injured;
import world.Path;
import world.Robot;
import world.RobotPercepcion;

public class RobotControl extends AbstractRobotControl{

	ArrayList<Injured> goodCons = new ArrayList<>();
	ArrayList<Injured> badCons 	= new ArrayList<>();
	ArrayList<Injured> deads 	= new ArrayList<>();
	
	private void refreashInjuredStatuses() {
		
		goodCons 	= new ArrayList<>();
		badCons 	= new ArrayList<>();
		deads 		= new ArrayList<>();
		
		for (Injured injured : percepcion.getDiscoveredInjureds()) {
			if(injured.isSaved()) {
				
			}
			else if (!injured.isAlive()) {
				deads.add(injured);
			}
			else if (injured.getHealth() < 60) {
				badCons.add(injured);
			}
			else {
				goodCons.add(injured);
			}
		}
	}
	
    public RobotControl(Robot robot, RobotPercepcion percepcion) {
        super(robot, percepcion);
    }
    
    public Integer step() {  
    	Cell location = robot.getLocation();
    	refreashInjuredStatuses();
    	if(robot.hasInjured()) {
    		return percepcion.getShortestExitPath(location).getFirstCell().directionFrom(location);
    	}
    	else if(!badCons.isEmpty()) {
    		for (Injured injured : badCons) {
    			if(injured.getLocation() != null) {
            		Path path = AStarSearch.search(location,injured.getLocation(), -1);
            		if(path != null) return path.getFirstCell().directionFrom(location);
    			}
			}
    	}
    	Path path = percepcion.getShortestUnknownPath(location);
    	if(path != null) {
        	return path.getFirstCell().directionFrom(location);
    	}else {
    		return percepcion.getShortestInjuredPath(location).getFirstCell().directionFrom(location);
    	}
    	
    }
}
```

## 5. feladat forráskódja:

```java
```
