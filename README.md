ST10518855part2POE
ST10518855part2POE
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */
package com.mycompany.main;

import java.util.Scanner;
import java.util.Random;
import java.util.ArrayList;
import java.util.List;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.charset.StandardCharsets;

public class Main {

    public static void main(String[] args) {

        Scanner scanner = new Scanner(System.in);

        System.out.println("========================");
        System.out.println("ChitChat Registration");
        System.out.println("========================");

        System.out.print("Enter your first name: ");
        String firstName = scanner.nextLine();

        System.out.print("Enter your last name: ");
        String lastName = scanner.nextLine();

        System.out.print("Enter a username: ");
        String username = scanner.nextLine();

        System.out.print("Enter a password: ");
        String password = scanner.nextLine();

        System.out.print("Enter your South African phone number (+27...): ");
        String cellPhoneNumber = scanner.nextLine();

        Login login = new Login(
                username, password, cellPhoneNumber,
                firstName, lastName
        );

        String registrationResult = login.registerUser();
        System.out.println(registrationResult);

        if (!registrationResult.equals("Registration successful.")) {
            System.out.println("Registration failed.");
            scanner.close();
            return;
        }

        // Login
        System.out.println("\n========================");
        System.out.println("LOGIN");
        System.out.println("========================");

        System.out.print("Enter your username: ");
        String enteredUsername = scanner.nextLine();

        System.out.print("Enter your password: ");
        String enteredPassword = scanner.nextLine();

        System.out.println(login.returnLoginStatus(
                enteredUsername, enteredPassword
        ));

        if (!login.loginUser(enteredUsername, enteredPassword)) {
            scanner.close();
            return;
        }

        System.out.println("\nWelcome to ChitChat.");

        // Ask for the message limit
        int requiredMessages;

        while (true) {
            System.out.print("How many messages would you like to send? ");

            try {
                requiredMessages = Integer.parseInt(scanner.nextLine());

                if (requiredMessages >= 0) {
                    break;
                }

                System.out.println(
                        "Please enter zero or a positive number."
                );

            } catch (NumberFormatException e) {
                System.out.println(
                        "Please enter a valid whole number."
                );
            }
        }

        boolean running = true;

        // Main menu
        while (running) {

            System.out.println("\n========================");
            System.out.println("ChitChat Menu");
            System.out.println("========================");
            System.out.println("1. Send Messages");
            System.out.println("2. Show recently sent messages");
            System.out.println("3. Quit");
            System.out.print("Choose an option: ");

            String choice = scanner.nextLine();

            switch (choice) {

                case "1":
                            
                    if (Message.returnTotalMessages()
                            >= requiredMessages) {
                        System.out.println(
                                "You have reached your message limit."
                        );
                        break;
                    }

                    System.out.print(
                            "Enter recipient cellphone number (+27...): "
                    );
                    String recipient = scanner.nextLine();

                    System.out.print("Enter your message: ");
                    String messageText = scanner.nextLine();

                    Message message = new Message(
                            recipient, messageText
                    );

                    if (!message.checkMessageID()) {
                        System.out.println("Invalid message ID.");
                        break;
                    }

                    String recipientResult =
                            message.checkRecipientCell();

                    System.out.println(recipientResult);

                    if (!message.isValidRecipientCell()) {
                        break;
                    }

                    if (!message.checkMessageLength()) {
                        System.out.println(
                                "Please enter a message of 250 characters or fewer."
                        );
                        break;
                    }

                    System.out.println(
                            "Message ID: " + message.getMessageID()
                    );
                    System.out.println(
                            "Message Hash: " + message.getMessageHash()
                    );

                    System.out.println("\n1. Send Message");
                    System.out.println("2. Disregard Message");
                    System.out.println("3. Store Message to send later");
                    System.out.print("Choose an option: ");

                    String messageChoice = scanner.nextLine();

                    String result =
                          message.processMessageChoice(messageChoice);

                    System.out.println(result);

                    if (message.wasSent()) {
                        System.out.println("\nMessage Details");
                        System.out.println("------------------------");
                        System.out.println(
                                "Message ID: " + message.getMessageID()
                        );
                        System.out.println(
                                "Message Hash: " + message.getMessageHash()
                        );
                        System.out.println(
                                "Recipient: " + message.getRecipient()
                        );
                        System.out.println(
                                "Message: " + message.getMessageText()
                        );
                    }

                    break;

                case "2":

                    System.out.println(Message.printMessages());
                    break;

                case "3":

                    running = false;
                    System.out.println(
                            "Thank you for using ChitChat."
                    );
                    break;

                default:

                    System.out.println(
                            "Invalid option. Please choose 1, 2 or 3."
                    );
            }
        }

        System.out.println("\nTotal messages sent: "
                + Message.returnTotalMessages());

        scanner.close();
    }
}
  


// =====================================================
// Login class
// =====================================================

class Login {

    private String username;
    private String password;
    private String cellPhoneNumber;
    private String firstName;
    private String lastName;

    // Constructor
    public Login(
            String username,
            String password,
            String cellPhoneNumber,
            String firstName,
            String lastName) {

        this.username = username;
        this.password = password;
        this.cellPhoneNumber = cellPhoneNumber;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // Check if username contains an underscore
    // and is no more than 5 characters long
    public boolean checkUserName() {

        return this.username.contains("_")
                && this.username.length() <= 5;
    }

  
    // Check password complexity
    public boolean checkPasswordComplexity() {

        boolean hasCapital = false;
        boolean hasNumber = false;
        boolean hasSpecial = false;

        // Password must have at least 8 characters
        if (password.length() < 8) {
            return false;
        }

        // Check every character in the password
        for (int i = 0; i < password.length(); i++) {

            char character = password.charAt(i);

            // Check for uppercase letter
            if (Character.isUpperCase(character)) {
                hasCapital = true;
            }

            // Check for number
            if (Character.isDigit(character)) {
                hasNumber = true;
            }

            // Check for special character
            if (!Character.isLetterOrDigit(character)) {
                hasSpecial = true;
            }
        }

        return hasCapital && hasNumber && hasSpecial;
    }

    // Check South African cellphone number
    public boolean checkCellPhoneNumber() {

        String regex = "^\\+27[6-8][0-9]{8}$";

        return cellPhoneNumber.matches(regex);
    }

    // Register user
    public String registerUser() {

        if (!checkUserName()) {

            return "Username is not correctly formatted; "
                    + "please ensure that your username contains "
                    + "an underscore and is no more than five "
                    + "characters in length.";
        }

        if (!checkPasswordComplexity()) {

            return "Password is not correctly formatted; "
                    + "please ensure that the password contains "
                    + "at least 8 characters, a capital letter, "
                    + "a number, and a special character.";
        }

        if (!checkCellPhoneNumber()) {

            return "Cell phone number is incorrectly formatted "
                    + "or does not contain the correct "
                    + "international code.";
        }

        return "Registration successful.";
    }

    // Check username and password during login
    public boolean loginUser(
            String enteredUsername,
            String enteredPassword) {

        return enteredUsername.equals(username)
                && enteredPassword.equals(password);
    }

    // Return login status
    public String returnLoginStatus(
            String enteredUsername,
            String enteredPassword) {

        if (loginUser(enteredUsername, enteredPassword)) {

            return "Welcome "
                    + firstName
                    + " "
                    + lastName
                    + ", it is great to see you again.";

        } else {

            return "Username or password incorrect, "
                    + "please try again.";
        }
    }
}
 
// ========================================
// Message Class 
// ========================================

class Message {

private String messageID;
    private String recipient;
    private String messageText;
    private String messageHash;

    private boolean sent;
    private boolean stored;

    private static int totalMessagesSent = 0;

    private static final List<Message> SENT_MESSAGES =
            new ArrayList<>();

    private static final Random RANDOM = new Random();

    // Constructor
    public Message(String recipient, String messageText) {

        this.messageID = generateMessageID();
        this.recipient = recipient;
        this.messageText = messageText;

        this.sent = false;
        this.stored = false;

        this.messageHash = createMessageHash();
    }

    // Generate a random ten-digit message ID
    private String generateMessageID() {

        long number = 1_000_000_000L
                + (long) (RANDOM.nextDouble()
                * 9_000_000_000L);

        return String.valueOf(number);
    }

    // Check that the message ID contains ten digits
    public boolean checkMessageID() {

        return messageID != null
                && messageID.matches("[0-9]{10}");
    }

    // Validate recipient cellphone number
    public String checkRecipientCell() {

        if (isValidRecipientCell()) {
            return "Cell phone number successfully captured.";
        }

        return "Cell phone number is incorrectly formatted. "
                + "Please include the South African international "
                + "code (+27) and try again.";
    }

    // Check recipient number format
    public boolean isValidRecipientCell() {

        String regex = "^\\+27[6-8][0-9]{8}$";

        return recipient != null
                && recipient.matches(regex);
    }

    // Check message length (maximum 250 characters)
    public boolean checkMessageLength() {

        return messageText != null
                && messageText.length() <= 250;
    }

    // Generate message hash
    public String createMessageHash() {

        String firstWord = "";
        String lastWord = "";

        if (messageText != null
                && !messageText.trim().isEmpty()) {

            String[] words = messageText.trim().split("\\s+");

            firstWord = words[0];
            lastWord = words[words.length - 1];
        }

        String firstTwoDigits = messageID.substring(0, 2);

        messageHash = (
                firstTwoDigits
                + ":" + (totalMessagesSent + 1)
                + ":" + firstWord + lastWord
        ).toUpperCase();

        return messageHash;
    }

    // Send, disregard or store a message
    public String processMessageChoice(String choice) {

        switch (choice) {

            case "1":

                sent = true;
                stored = false;

                totalMessagesSent++;

                SENT_MESSAGES.add(this);

                // Update hash using the current message count
                createMessageHash();

                return "Message successfully sent.";

            case "2":

                sent = false;
                stored = false;

                return "Message disregarded.";

            case "3":

                sent = false;
                stored = true;

                if (storeMessage()) {
                    return "Message successfully stored.";
                }

                stored = false;

                return "Message could not be stored.";

            default:

                return "Invalid message option.";
        }
    }

    // Store message details in messages.json
    public boolean storeMessage() {

        Path path = Path.of("messages.json");

        try {

            String jsonObject = "{"
                    + "\"messageID\":\""
                    + escapeJSON(messageID) + "\","
                    + "\"messageHash\":\""
                    + escapeJSON(messageHash) + "\","
                    + "\"recipient\":\""
                    + escapeJSON(recipient) + "\","
                    + "\"message\":\""
                    + escapeJSON(messageText)
                    + "\"}";

            String existing = "[]";

            if (Files.exists(path)) {

                existing = Files.readString(
                        path,
                        StandardCharsets.UTF_8
                ).trim();
            }

            if (existing.isEmpty()) {
                existing = "[]";
            }

            // Check that the existing file contains a JSON array
            if (!existing.startsWith("[")
                    || !existing.endsWith("]")) {

                throw new IOException(
                        "The existing JSON file is invalid."
                );
            }

            // Remove the closing bracket
            String contents = existing.substring(
                    0,
                    existing.length() - 1
            );

            // Add the new message
            if (contents.trim().equals("[")) {
                contents += jsonObject;
            } else {
                contents += "," + jsonObject;
            }

            contents += "]";

            Files.writeString(
                    path,
                    contents,
                    StandardCharsets.UTF_8
            );

            return true;

        } catch (IOException e) {

            System.out.println(
                    "Error storing message: " + e.getMessage()
            );

            return false;
        }
    }

    // Escape special characters in JSON text
    private String escapeJSON(String text) {

        if (text == null) {
            return "";
        }

        return text.replace("\\", "\\\\")
                .replace("\"", "\\\"")
                .replace("\n", "\\n")
                .replace("\r", "\\r")
                .replace("\t", "\\t");
    }

    // Display all sent messages
    public static String printMessages() {

        if (SENT_MESSAGES.isEmpty()) {
            return "No messages have been sent.";
        }

        StringBuilder output = new StringBuilder();

        for (Message message : SENT_MESSAGES) {

            output.append("Message ID: ")
                    .append(message.messageID)
                    .append("\n");

            output.append("Message Hash: ")
                    .append(message.messageHash)
                    .append("\n");

            output.append("Recipient: ")
                    .append(message.recipient)
                    .append("\n");

            output.append("Message: ")
                    .append(message.messageText)
                    .append("\n");

            output.append("------------------------")
                    .append("\n");
        }

        return output.toString();
    }

    // Return total number of messages sent
    public static int returnTotalMessages() {

        return totalMessagesSent;
    }

    // Getters
    public String getMessageID() {
        return messageID;
    }

    public String getRecipient() {
        return recipient;
    }

    public String getMessageText() {
        return messageText;
    }

    public String getMessageHash() {
        return messageHash;
    }

    public boolean wasSent() {
        return sent;
    }

    public boolean wasStored() {
        return stored;
    }
}
 





    









