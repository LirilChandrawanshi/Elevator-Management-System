# Elevator-Management-System
This project simulates an elevator management system using Java and multi-threading. The system can manage multiple elevators, handle floor requests, and assign the nearest available elevator to a request.
import java.util.ArrayList;
import java.util.List;

class Elevator implements Runnable {
    private int currentFloor;
    private int targetFloor;
    private boolean movingUp;

    public Elevator(int startFloor) {
        this.currentFloor = startFloor;
        this.targetFloor = startFloor;
        this.movingUp = true;
    }

    public synchronized void moveToFloor(int floor) {
        this.targetFloor = floor;
        notify();
    }

    @Override
    public void run() {
        while (true) {
            synchronized (this) {
                while (currentFloor == targetFloor) {
                    try {
                        wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            if (currentFloor < targetFloor) {
                moveUp();
            } else {
                moveDown();
            }
        }
    }

    private void moveUp() {
        currentFloor++;
        System.out.println("Elevator moving up to floor: " + currentFloor);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private void moveDown() {
        currentFloor--;
        System.out.println("Elevator moving down to floor: " + currentFloor);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class ElevatorController {
    private final List<Elevator> elevators;

    public ElevatorController(int numElevators) {
        elevators = new ArrayList<>();
        for (int i = 0; i < numElevators; i++) {
            Elevator elevator = new Elevator(0);
            Thread thread = new Thread(elevator);
            thread.start();
            elevators.add(elevator);
        }
    }

    public void requestElevator(int floor) {
        Elevator nearestElevator = elevators.get(0);
        for (Elevator elevator : elevators) {
            if (Math.abs(elevator.getCurrentFloor() - floor) < Math.abs(nearestElevator.getCurrentFloor() - floor)) {
                nearestElevator = elevator;
            }
        }
        nearestElevator.moveToFloor(floor);
    }
}

public class ElevatorSystem {
    public static void main(String[] args) {
        ElevatorController controller = new ElevatorController(3);

        controller.requestElevator(5);
        controller.requestElevator(2);
        controller.requestElevator(8);
    }
}
