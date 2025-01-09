import javax.swing.*;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

class Elevator implements Runnable {
    private int currentFloor;
    private int targetFloor;
    private final JLabel displayLabel;
    private boolean moving;

    public Elevator(int startFloor, JLabel displayLabel) {
        this.currentFloor = startFloor;
        this.targetFloor = startFloor;
        this.displayLabel = displayLabel;
        this.moving = false;
    }

    public synchronized int getCurrentFloor() {
        return currentFloor;
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
        moving = true;
        updateDisplay();
        currentFloor++;
        sleep();
        moving = false;
        updateDisplay();
    }

    private void moveDown() {
        moving = true;
        updateDisplay();
        currentFloor--;
        sleep();
        moving = false;
        updateDisplay();
    }

    private void updateDisplay() {
        SwingUtilities.invokeLater(() -> {
            displayLabel.setText("Floor: " + currentFloor);
            displayLabel.setBackground(moving ? Color.YELLOW : Color.WHITE);
            displayLabel.setOpaque(true);
        });
    }

    private void sleep() {
        try {
            Thread.sleep(1000); // Simulates the time taken to move between floors
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class ElevatorController {
    private final List<Elevator> elevators;

    public ElevatorController(int numElevators, List<JLabel> displayLabels) {
        elevators = new ArrayList<>();
        for (int i = 0; i < numElevators; i++) {
            Elevator elevator = new Elevator(0, displayLabels.get(i));
            Thread thread = new Thread(elevator);
            thread.start();
            elevators.add(elevator);
        }
    }

    public void requestElevator(int floor) {
        if (floor < 0 || floor > 10) { // Assuming a building with 10 floors
            JOptionPane.showMessageDialog(null, "Invalid floor number. Please enter a floor between 0 and 10.", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }
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
        SwingUtilities.invokeLater(ElevatorSystem::createAndShowGUI);
    }

    private static void createAndShowGUI() {
        JFrame frame = new JFrame("Elevator System");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 600);

        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(4, 1));

        List<JLabel> elevatorLabels = new ArrayList<>();

        // Add labels for elevators
        for (int i = 0; i < 3; i++) {
            JPanel elevatorPanel = new JPanel();
            elevatorPanel.setLayout(new BorderLayout());

            JLabel label = new JLabel("Floor: 0", SwingConstants.CENTER);
            elevatorLabels.add(label);  // Add the label to the list
            label.setBorder(BorderFactory.createLineBorder(Color.BLACK));
            label.setBackground(Color.WHITE);
            label.setOpaque(true);

            elevatorPanel.add(new JLabel("Elevator " + (i + 1), SwingConstants.CENTER), BorderLayout.NORTH);
            elevatorPanel.add(label, BorderLayout .CENTER);

            panel.add(elevatorPanel);
        }

        ElevatorController controller = new ElevatorController(3, elevatorLabels);

        JPanel requestPanel = new JPanel();
        requestPanel.setLayout(new FlowLayout());

        // Create floor buttons
        for (int i = 0; i <= 10; i++) {
            JButton floorButton = new JButton(String.valueOf(i));
            floorButton.addActionListener(e -> controller.requestElevator(Integer.parseInt(floorButton.getText())));
            requestPanel.add(floorButton);
        }

        frame.add(panel, BorderLayout.CENTER);
        frame.add(requestPanel, BorderLayout.SOUTH);

        frame.setVisible(true);
    }
}
