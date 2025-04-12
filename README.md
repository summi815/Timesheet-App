# Timesheet-App
This is a Java-based Timesheet Management System developed using Swing for GUI. It allows users to log in securely, start and end tasks, and track the time spent on each task.
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

class User {
    private String username;
    private String password;
    private TimeSheet timeSheet;

    public User(String username, String password) {
        this.username = username;
        this.password = password;
        this.timeSheet = new TimeSheet();
    }

    public String getUsername() {
        return username;
    }

    public boolean checkPassword(String password) {
        return this.password.equals(password);
    }

    public TimeSheet getTimeSheet() {
        return timeSheet;
    }
}

class Task {
    private String taskName;
    private LocalDateTime startTime;
    private LocalDateTime endTime;

    public Task(String taskName) {
        this.taskName = taskName;
        this.startTime = LocalDateTime.now();
        this.endTime = null;
    }

    public String getTaskName() {
        return taskName;
    }

    public void endTask() {
        this.endTime = LocalDateTime.now();
    }

    public Duration getTaskDuration() {
        if (endTime == null) {
            return Duration.ZERO; // Task has not ended yet
        }
        return Duration.between(startTime, endTime);
    }

    public boolean isOngoing() {
        return endTime == null;
    }
}

class TimeSheet {
    private List<Task> tasks;

    public TimeSheet() {
        this.tasks = new ArrayList<>();
    }

    public void startNewTask(String taskName) {
        Task task = new Task(taskName);
        tasks.add(task);
    }

    public void endTask(String taskName) {
        for (Task task : tasks) {
            if (task.getTaskName().equals(taskName) && task.isOngoing()) {
                task.endTask();
                return;
            }
        }
    }

    public String displayTasks() {
        StringBuilder sb = new StringBuilder();
        sb.append("Task List:\n");
        for (Task task : tasks) {
            sb.append("Task: ").append(task.getTaskName())
              .append(", Duration: ").append(task.getTaskDuration().toMinutes()).append(" minutes\n");
        }
        return sb.toString();
    }
}

public class TimesheetAppGUI {
    private static User user = new User("Aryan", "114488");

    public static void main(String[] args) {
        JFrame frame = new JFrame("Timesheet Application");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 300);
        
        JPanel panel = new JPanel();
        panel.setLayout(new CardLayout());

        // Login Panel
        JPanel loginPanel = new JPanel();
        loginPanel.setLayout(new FlowLayout(FlowLayout.CENTER, 10, 10));

        JLabel usernameLabel = new JLabel("Username: ");
        JTextField usernameField = new JTextField(15);
        JLabel passwordLabel = new JLabel("Password: ");
        JPasswordField passwordField = new JPasswordField(15);
        JButton loginButton = new JButton("Login");
        JLabel errorLabel = new JLabel("", SwingConstants.CENTER);
        
        loginPanel.add(usernameLabel);
        loginPanel.add(usernameField);
        loginPanel.add(passwordLabel);
        loginPanel.add(passwordField);
        loginPanel.add(loginButton);
        loginPanel.add(errorLabel);

        panel.add(loginPanel, "Login");

        // Task Management Panel
        JPanel taskPanel = new JPanel();
        taskPanel.setLayout(new GridLayout(5, 1));

        JTextArea taskArea = new JTextArea();
        taskArea.setEditable(false);
        taskArea.setText("Welcome! Please login to start.");
        
        JTextField taskNameField = new JTextField();
        JButton startTaskButton = new JButton("Start Task");
        JButton endTaskButton = new JButton("End Task");
        JButton viewTasksButton = new JButton("View Tasks");
        JButton logoutButton = new JButton("Logout");

        taskPanel.add(new JScrollPane(taskArea));
        taskPanel.add(taskNameField);
        taskPanel.add(startTaskButton);
        taskPanel.add(endTaskButton);
        taskPanel.add(viewTasksButton);
        taskPanel.add(logoutButton);
        
        panel.add(taskPanel, "Tasks");

        // Handle Login Button Action
        loginButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String username = usernameField.getText();
                String password = new String(passwordField.getPassword());

                if (user.getUsername().equals(username) && user.checkPassword(password)) {
                    ((CardLayout) panel.getLayout()).show(panel, "Tasks");
                } else {
                    errorLabel.setText("Invalid username or password!");
                }
            }
        });

        // Handle Start Task Button Action
        startTaskButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String taskName = taskNameField.getText();
                if (!taskName.isEmpty()) {
                    user.getTimeSheet().startNewTask(taskName);
                    taskArea.append("\nStarted task: " + taskName);
                    taskNameField.setText("");
                } else {
                    JOptionPane.showMessageDialog(frame, "Task name cannot be empty!");
                }
            }
        });

        // Handle End Task Button Action
        endTaskButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String taskName = taskNameField.getText();
                if (!taskName.isEmpty()) {
                    user.getTimeSheet().endTask(taskName);
                    taskArea.append("\nEnded task: " + taskName);
                    taskNameField.setText("");
                } else {
                    JOptionPane.showMessageDialog(frame, "Task name cannot be empty or task may not have started!");
                }
            }
        });

        // Handle View Tasks Button Action
        viewTasksButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                taskArea.setText(user.getTimeSheet().displayTasks());
            }
        });

        // Handle Logout Button Action
        logoutButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                ((CardLayout) panel.getLayout()).show(panel, "Login");
                usernameField.setText("");
                passwordField.setText("");
                errorLabel.setText("");
                taskArea.setText("Welcome! Please login to start.");
            }
        });

        frame.add(panel);
        frame.setVisible(true);
    }
}
