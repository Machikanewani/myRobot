import robocode.*;
import java.awt.Color;
import java.awt.geom.Point2D; 

public class G8LeaderRobot extends AdvancedRobot{
    double attackRange;
    double fieldXMin, fieldXMax, fieldYMin, fieldYMax, maxLength;
    int state = 0, dir = 0;
    boolean gunTurning = false, interruptible = true;
    double x,y; //used in predictive shooting meaning enemy's (x,y)


    public void run(){
        setBodyColor(new Color(90, 150, 75));
        setGunColor(new Color(90, 150, 75));
        setRadarColor(new Color(200, 55, 70));
        setBulletColor(new Color(255, 255, 0));
        setScanColor(new Color(255, 255, 0));

        setAdjustGunForRobotTurn(true);
        //Substantial field size considering the size of the robot
        fieldXMin = getWidth() / 2;
        fieldXMax = getBattleFieldWidth() - fieldXMin;
        fieldYMin = getHeight() / 2;
        fieldYMax = getBattleFieldHeight() - fieldYMin;
        maxLength = Math.max(fieldXMax - fieldXMin, fieldYMax - fieldYMin);

        attackRange = maxLength / 2;

        double degree = 0, smallest = getX() - fieldXMin;
        boolean isGunTurnLeft = false, goStraight = true;

        if(smallest > getY() - fieldYMin){
            smallest = getY();
            dir = 1;
        }
        if(smallest > fieldXMax - getX()){
            smallest = fieldXMax - getX();
            dir = 2;
        }
        if(smallest > fieldYMax - getY()){
            smallest = fieldYMax - getY();
            dir = 3;
        }

        switch(dir){
            case 0:
                degree = (getHeading() + 90) % 360;
                break;
            case 1:
                degree = (getHeading() + 180) % 360;
                break;
            case 2:
                degree = (getHeading() + 270) % 360;
                break;
            case 3:
                degree = getHeading();
                break;
            default:
                break;
        }

        if(degree > 180){
            turnLeft(degree - 360);
        }else{
            turnLeft(degree);
        }
        gunTurning = true;
        if(getGunHeading() - getHeading() > 180){
            turnGunLeft(getGunHeading() - getHeading() - 360);
        }else if(getGunHeading() - getHeading() < -180){
            turnGunLeft(getGunHeading() - getHeading() + 360);
        }else{
            turnGunLeft(getGunHeading() - getHeading());
        }
        gunTurning = false;
        forward(smallest);

        state = 1;
        if(dir == 0){
            dir = 3;
        }else{
            dir--;
        }
        while(true){
            turnLeft(90);
            gunTurning = true;
            turnGunLeft(90);
            gunTurning = false;
            if(dir == 3){
                dir = 0;
            }else{
                dir++;
            }
            goStraight = true;
            while(goStraight){
                if(getGunTurnRemaining() == 0){
                    isGunTurnLeft = !isGunTurnLeft;
                    if(isGunTurnLeft){
                        setTurnGunLeft(90);
                    }else{
                        setTurnGunRight(90);
                    }
                    execute();
                }
                forward(maxLength/8);

                goStraight = !isNearbyWall(10);
            }
        }
    }

    public void onHitByBullet(HitByBulletEvent e){
        double bulletDir = e.getBearing();
        if(!interruptible || e.getName().contains("G8")){
            return;
        }else if(bulletDir > -105 && bulletDir < -75){
            backward(50);
        }else{
            dodge();
        }
    }

    public void forward(double dis){  // go forwards and avoid hitting walls
        double robotX, robotY;
        double heading;

        robotX = getX();
        robotY = getY();
        heading = getHeading();

        switch((int)heading){
            case 90:          // rightward
              if(dis < (fieldXMax - robotX)) ahead(dis);
              else ahead(fieldXMax - robotX -1);
              break;

            case 180:         //downward
              if(dis < (robotY - fieldYMin)) ahead(dis);
              else ahead(robotY - fieldYMin - 1);
              break;

            case 270:         //leftward
              if(dis < (robotX - fieldXMin)) ahead(dis);
              else ahead(robotX - fieldXMin - 1);
              break;

            case 0:           //upward
            case 360:
              if(dis < (fieldYMax - robotY)) ahead(dis);
              else ahead(fieldYMax - robotY - 1);
              break;
        }

    }

    public void backward(double dis){ //go backwards and avoid hitting walls
        double robotX, robotY;
        double heading;

        robotX = getX();
        robotY = getY();
        heading = getHeading();

        switch((int)heading){
            case 270:         // rightward
              if(dis < (fieldXMax - robotX)) back(dis);
              else back(fieldXMax - robotX -1);
              break;

            case 0:         //downward
            case 360:
              if(dis < (robotY - fieldYMin)) back(dis);
              else back(fieldYMax - robotY - 1);
              break;

            case 90:         //leftward
              if(dis < (robotX - fieldXMin)) back(dis);
              else back(robotX - fieldXMin - 1);
              break;

            case 180:           //upward
              if(dis < (fieldYMax - robotY)) back(dis);
              else back(fieldYMax - robotY - 1);
              break;
        }
    }

    public void dodge(){   //go round to dodge enemy and attack
        if(!interruptible || isNearbyWall(10)) return; 
        processRemaining();
        interruptible = false;
        stop(true);
        turnLeft(90);
        forward(50);
        turnRight(90);
        forward(100);
        if(isNearbyWall(5)){
            interruptible = true;
            return;
        }
        turnRight(90);
        forward(50);
        turnLeft(90);
        interruptible = true;
    }

    public void shoot(ScannedRobotEvent e){ // fire
        if(getGunHeat() != 0)return;
        fire(getfirePower(e));

    }

    public void onScannedRobot(ScannedRobotEvent e) {
        String enemyName;
        double robotBearing;
        double enemyHeading;

        double enemyDistance;
        double enemyVelocity;
        double bulletSpeed;
        int flag = 0;

        enemyName = e.getName();
        robotBearing = Math.abs(e.getBearing());
        enemyHeading = e.getHeading();

        enemyDistance = e.getDistance(); 
        enemyVelocity = e.getVelocity(); 
        bulletSpeed = 20 - getfirePower(e) * 3;
        long time = (long)(enemyDistance / bulletSpeed);
        double futureX = getFutureX(e,time);
        double futureY = getFutureY(e,time); 
        double futureDe = futureBearing(getX(), getY(), futureX, futureY);
        double turnedDe = 0;
        double gunTurnRemaining = getGunTurnRemaining();

        if(enemyName.contains("G8")){
            if(interruptible && robotBearing < 5 && e.getDistance() < 400){
                backward(100);
            }
            return;
        }

        if(state == 0){
            shoot(e);
            if(!gunTurning && interruptible)dodge();
        } else {
            if(getfirePower(e) >= 0.1){
                gunTurning = true;
                if(interruptible){
                    interruptible = !interruptible;
                    flag = 1;
                }
                turnedDe = normalizeBearing(futureDe - getGunHeading());
				
                if(getHeading() == 180){
					if(getGunHeading() + turnedDe > 180) turnedDe = 180 - getGunHeading();
				}
                turnGunRight(turnedDe);
                shoot(e);
                turnGunLeft(turnedDe);
                turnGunRight(gunTurnRemaining);
                gunTurning = false;
            }

            if(flag == 1){
                interruptible = !interruptible;
                flag = 0;
            }

            if(gunTurning || !interruptible)return;
            if(enemyName.contains("Walls") && robotBearing < 5){
                if(Math.abs(Math.abs(getHeading() - enemyHeading)%180) < 5){
                    //被弾するまでWallsを打ち続ける
                    backward(50);
                }else if(Math.abs(Math.abs(getHeading() - enemyHeading)%180 - 90) < 5){
				   interruptible =true;
                   dodge();
				 }  
            }else if(robotBearing < 5 && Math.abs(Math.abs(getHeading() - enemyHeading) - 180) < 5) dodge();
        }
    }

    private boolean isNearbyWall(double decisionDistance){
        switch(dir){
            case 0:
                if(Math.abs(fieldYMin - getY()) < decisionDistance){
                    return true;
                }
                break;
            case 1:
                if(Math.abs(fieldXMax - getX()) < decisionDistance){
                    return true;
                }
                break;
            case 2:
                if(Math.abs(fieldYMax - getY()) < decisionDistance){
                    return true;
                }
                break;
            case 3:
                if(Math.abs(fieldXMin - getX()) < decisionDistance){
                    return true;
                }
                break;
            default:
                break;
        }
        return false;
    }

    public void onHitRobot(HitRobotEvent e) {
      double distanceRemain = getDistanceRemaining();
      // If he's in front of us, set back up a bit.
      if (e.isMyFault()) {
        processRemaining();
        backward(50);
        if (interruptible)dodge();
        forward(50 + distanceRemain);
      } // else he's in back of us, so set ahead a bit.
      else {
        ;
      }
    }

    private void processRemaining(){
        turnRight(getTurnRemaining());
        turnGunRight(getGunTurnRemaining());
        stop(true);
    }

    private double getFutureX(ScannedRobotEvent e, long when){ 
        double absBearingDe = getHeading() + e.getBearing();
        if (absBearingDe < 0) absBearingDe += 360;

        x = getX() + Math.sin(Math.toRadians(absBearingDe)) * e.getDistance();

        return x + Math.sin(Math.toRadians(e.getHeading())) * e.getVelocity() * when;

    }

    private double getFutureY(ScannedRobotEvent e, long when){
        double absBearingDe = getHeading() + e.getBearing();
        if (absBearingDe < 0) absBearingDe += 360;

        y = getY() + Math.cos(Math.toRadians(absBearingDe)) * e.getDistance();

        return y + Math.cos(Math.toRadians(e.getHeading())) * e.getVelocity() * when;
    }

    private double futureBearing(double robotX, double robotY, double enemyX, double enemyY){
        double dx = enemyX - robotX;
        double dy = enemyY - robotY;
        double hyp = Point2D.distance(robotX,robotY,enemyX,enemyY);
        double arcSin = Math.toDegrees(Math.asin(dx / hyp));

        if(dx > 0 && dy > 0) return arcSin;
        else if(dx < 0 && dy > 0) return 360 + arcSin;
        else if(dx > 0 && dy < 0) return 180 - arcSin;
        else if(dx < 0 && dy < 0) return 190 - arcSin;

        return 0;
    }

    private double normalizeBearing(double angle){
        while (angle >  180) angle -= 360;
        while (angle < -180) angle += 360;
        return angle;
    }

    private double getfirePower(ScannedRobotEvent e){
        double distance;
        double enemyHeading;
        double robotBearing;
        double robotHeading;
        double enemyVelocity;
        float multi;
        double power = 0;
        String enemyName;

        distance = e.getDistance();
        enemyName = e.getName();
        enemyHeading = e.getHeading();   // enemy's heading
        robotBearing = e.getBearing();      //  enemy's angle from robot (bearing)
        robotHeading = getHeading();
        enemyVelocity = e.getVelocity(); //enemy's velocity

        if((Math.abs(robotHeading + (180 - enemyHeading)) > 5 || Math.abs((180 - robotHeading) + enemyHeading) > 5) //not heading toward here
             && enemyVelocity <= 0.3                  //velocity is almost 0 (0.3 can be changed)
               && enemyName.contains("Leader")) multi = 3;    // is leader
        else if ((Math.abs(robotHeading + (180 - enemyHeading)) > 5 || Math.abs((180 - robotHeading) + enemyHeading) > 5) && enemyVelocity <= 0.3) multi = 2;
                                                      // not heading toward here AND V is almost 0
        else if (enemyName.contains("Leader")) multi = (float)1.5;     //is leader
        else multi = 1;

        if(enemyName.contains("Walls") && Math.abs(robotBearing) < 5){ //Walls 
            power = 3;
        }
        else if(distance < attackRange){  //others 
            power = 3.0 * (1.0 - Math.pow((distance / attackRange), 3.0)) * multi;
        }

        return power;
    }
}
