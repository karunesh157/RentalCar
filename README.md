import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.util.List;

class Car {
    private String carId;
    private String brand;
    private String model;
    private double basePricePerDay;
    private boolean isAvailable;

    public Car(String carId, String brand, String model, double basePricePerDay) {
        this.carId = carId;
        this.brand = brand;
        this.model = model;
        this.basePricePerDay = basePricePerDay;
        this.isAvailable = true;
    }

    public String getCarId() { return carId; }
    public String getBrand() { return brand; }
    public String getModel() { return model; }
    public boolean isAvailable() { return isAvailable; }
    public double calculatePrice(int rentalDays) { return basePricePerDay * rentalDays; }
    public void rent() { isAvailable = false; }
    public void returnCar() { isAvailable = true; }
}

class Customer {
    private String customerId;
    private String name;

    public Customer(String customerId, String name) {
        this.customerId = customerId;
        this.name = name;
    }

    public String getCustomerId() { return customerId; }
    public String getName() { return name; }
}

class Rental {
    private Car car;
    private Customer customer;
    private int days;

    public Rental(Car car, Customer customer, int days) {
        this.car = car;
        this.customer = customer;
        this.days = days;
    }

    public Car getCar() { return car; }
    public Customer getCustomer() { return customer; }
    public int getDays() { return days; }
}

public class CarRentalApp extends JFrame {
    private List<Car> cars = new ArrayList<>();
    private List<Customer> customers = new ArrayList<>();
    private List<Rental> rentals = new ArrayList<>();

    private JComboBox<String> carComboBox;
    private JTextField nameField, daysField;
    private JTextArea outputArea;

    public CarRentalApp() {
        setTitle("Car Rental System");
        setSize(600, 500);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        // Top panel for rental input
        JPanel topPanel = new JPanel(new GridLayout(3, 2, 5, 5));
        topPanel.setBorder(BorderFactory.createTitledBorder("Rent a Car"));
        topPanel.add(new JLabel("Customer Name:"));
        nameField = new JTextField();
        topPanel.add(nameField);

        topPanel.add(new JLabel("Select Car:"));
        carComboBox = new JComboBox<>();
        topPanel.add(carComboBox);

        topPanel.add(new JLabel("Rental Days:"));
        daysField = new JTextField();
        topPanel.add(daysField);

        add(topPanel, BorderLayout.NORTH);

        // Output area
        outputArea = new JTextArea();
        outputArea.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(outputArea);
        scrollPane.setBorder(BorderFactory.createTitledBorder("Output"));
        add(scrollPane, BorderLayout.CENTER);

        // Bottom panel for buttons
        JPanel bottomPanel = new JPanel();
        JButton rentButton = new JButton("Rent Car");
        JButton returnButton = new JButton("Return Car");
        bottomPanel.add(rentButton);
        bottomPanel.add(returnButton);
        add(bottomPanel, BorderLayout.SOUTH);

        // Button actions
        rentButton.addActionListener(e -> rentCar());
        returnButton.addActionListener(e -> returnCar());

        // Add some default cars
        addCar(new Car("C001", "Toyota", "Camry", 60.0));
        addCar(new Car("C002", "Honda", "Accord", 70.0));
        addCar(new Car("C003", "Mahindra", "Thar", 150.0));

        updateCarComboBox();
        setVisible(true);
    }

    private void addCar(Car car) {
        cars.add(car);
    }

    private void updateCarComboBox() {
        carComboBox.removeAllItems();
        for (Car car : cars) {
            if (car.isAvailable()) {
                carComboBox.addItem(car.getCarId() + " - " + car.getBrand() + " " + car.getModel());
            }
        }
    }

    private void rentCar() {
        String name = nameField.getText().trim();
        String selectedCar = (String) carComboBox.getSelectedItem();
        String daysText = daysField.getText().trim();

        if (name.isEmpty() || selectedCar == null || daysText.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Please fill all fields.");
            return;
        }

        int days;
        try {
            days = Integer.parseInt(daysText);
            if (days <= 0) throw new NumberFormatException();
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(this, "Enter a valid number of days.");
            return;
        }

        String carId = selectedCar.split(" - ")[0];
        Car car = null;
        for (Car c : cars) {
            if (c.getCarId().equals(carId) && c.isAvailable()) {
                car = c;
                break;
            }
        }

        if (car != null) {
            Customer customer = new Customer("CUS" + (customers.size() + 1), name);
            customers.add(customer);
            car.rent();
            rentals.add(new Rental(car, customer, days));

            double totalPrice = car.calculatePrice(days);
            outputArea.append("Rental Confirmed:\nCustomer: " + customer.getName() +
                    "\nCar: " + car.getBrand() + " " + car.getModel() +
                    "\nDays: " + days + String.format("\nTotal Price: $%.2f\n\n", totalPrice));
            updateCarComboBox();
        } else {
            JOptionPane.showMessageDialog(this, "Car not available.");
        }
    }

    private void returnCar() {
        String carId = JOptionPane.showInputDialog(this, "Enter Car ID to return:");
        if (carId == null || carId.trim().isEmpty()) return;

        Car carToReturn = null;
        for (Car car : cars) {
            if (car.getCarId().equals(carId.trim()) && !car.isAvailable()) {
                carToReturn = car;
                break;
            }
        }

        if (carToReturn != null) {
            Rental rentalToRemove = null;
            Customer customer = null;
            for (Rental rental : rentals) {
                if (rental.getCar() == carToReturn) {
                    rentalToRemove = rental;
                    customer = rental.getCustomer();
                    break;
                }
            }

            if (rentalToRemove != null && customer != null) {
                rentals.remove(rentalToRemove);
                carToReturn.returnCar();
                outputArea.append("Car returned successfully by " + customer.getName() + "\n\n");
                updateCarComboBox();
            } else {
                JOptionPane.showMessageDialog(this, "Rental information missing.");
            }
        } else {
            JOptionPane.showMessageDialog(this, "Invalid car ID or car not rented.");
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(CarRentalApp::new);
    }
}

