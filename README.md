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
package rescueagents;

import java.awt.Color;
import java.util.ArrayList;
import java.util.Vector;

import rescueframework.AbstractRobotControl;
import world.AStarSearch;
import world.Cell;
import world.Injured;
import world.Path;
import world.Robot;
import world.RobotPercepcion;

public class RobotControl extends AbstractRobotControl{
	
	private static Vector<Injured> injureds = new Vector<>();
	private static Vector<Robot> assigned	= new Vector<>();
	private static Vector<Integer> distace	= new Vector<>();
	private static Vector<InjuredStatus> status = new Vector<>();
	
	private static void refreshDatabase(RobotPercepcion percepcion) {
		ArrayList<Injured> injuredList = percepcion.getDiscoveredInjureds();
		for (Injured injured : injuredList) {
			if(injureds.contains(injured));
			else {
				injureds.add(injured);
				assigned.add(null);
				distace.add(null);
				status.add(null);
			}
		}
		for (int i = 0; i < injuredList.size(); i++) {
			Injured injured = injuredList.get(i);
			Cell injuredLocation = injured.getLocation();
			Integer exitDistance;
			if(injuredLocation != null) {
				Path exitPath = percepcion.getShortestExitPath(injuredLocation);
				if(exitPath != null) {
					exitDistance = exitPath.getLength();
					distace.set(i, exitDistance);
				}
				else exitDistance = null;
			}
			else exitDistance = null;
			if(status.get(i) != InjuredStatus.SAVING) {
				if(injured.getHealth()<=0) {
					if(status.get(i) != InjuredStatus.DEAD) {
						status.set(i, InjuredStatus.DEAD);
					}
				}
				else if(exitDistance != null && injured.getHealth() < exitDistance) {
					if(status.get(i) != InjuredStatus.UNSAVEABLE) {
						status.set(i, InjuredStatus.UNSAVEABLE);
					}
				}
			}
		}
	}
	
	private static Injured getCritical(Robot robot, RobotPercepcion percepcion) {
		Cell location = robot.getLocation();
		Injured worstSaveable = null;
		Integer worstDeltaTime = null;
		for (int i = 0; i < injureds.size(); i++) {
			if(status.get(i) == InjuredStatus.ALIVE) {
				Injured injured = injureds.get(i);
				Path path = null;
				if(injured.getLocation() != null)
					path = AStarSearch.search(location, injured.getLocation(), -1, Color.ORANGE);
				if(path != null && distace.get(i) != null) {
					Integer deltaTime = injured.getHealth() - (distace.get(i) + path.getLength());
					if((worstDeltaTime == null || worstDeltaTime < deltaTime) && !(deltaTime < 0)) {
						worstDeltaTime = deltaTime;
						worstSaveable = injured;
					}
				}
			}
		}
		if(worstDeltaTime != null && worstDeltaTime < 800 ) {
			Integer id =injureds.indexOf(worstSaveable);
			assigned.set(id, robot);
			status.set(id, InjuredStatus.SAVING);
			return worstSaveable;
		}
		return null;
	}
	
	private Injured selecetInjured = null;
	
    public RobotControl(Robot robot, RobotPercepcion percepcion) {
        super(robot, percepcion);
    }
    
    public Integer step() {
    	if(selecetInjured != null && selecetInjured.isSaved()) selecetInjured = null;
    	Cell location = robot.getLocation();
    	refreshDatabase(percepcion);
    	
    	if(robot.hasInjured()) {
    		return percepcion.getShortestExitPath(location).getFirstCell().directionFrom(location);
    	}
    	if(selecetInjured != null) {
    		return AStarSearch.search(location, selecetInjured.getLocation(), -1).getFirstCell().directionFrom(location);
        }
    	
    	Injured critical = getCritical(robot, percepcion);
    	if(critical != null) {
    		selecetInjured = critical;
    		return AStarSearch.search(location, selecetInjured.getLocation(), -1).getFirstCell().directionFrom(location);
    	}
    	
    	Path explorePath = percepcion.getShortestUnknownPath(location);
    	if(explorePath != null) {
    		return explorePath.getFirstCell().directionFrom(location);
        }
    	
    	Path injuredPath = percepcion.getShortestInjuredPath(location);
    	if(injuredPath != null) {
    		return injuredPath.getFirstCell().directionFrom(location);
    	}
    	return null;
    }
    
    private enum InjuredStatus{
    	SAVING,
    	ALIVE,
    	UNSAVEABLE,
    	DEAD
    }
    	
}
```
