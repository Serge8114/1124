import java.io.*;
import java.net.*;
import java.nio.charset.StandardCharsets;

public class Client {
    private static final String HOST = "localhost";
    private static final int PORT = 12345;

    public static void main(String[] args) {
        try (Socket socket = new Socket(HOST, PORT);
             BufferedInputStream bufferedInput = new BufferedInputStream(socket.getInputStream());
             BufferedOutputStream bufferedOutput = new BufferedOutputStream(socket.getOutputStream());
             InputStreamReader charInput = new InputStreamReader(bufferedInput, StandardCharsets.UTF_8);
             OutputStreamWriter charOutput = new OutputStreamWriter(bufferedOutput, StandardCharsets.UTF_8);
             BufferedReader in = new BufferedReader(charInput);
             PrintWriter out = new PrintWriter(charOutput, true);
             BufferedReader stdIn = new BufferedReader(new InputStreamReader(System.in))) {
            
            System.out.println("Подключено к серверу. Вводите строки (Ctrl+D/Ctrl+C для выхода):");
            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput);
                String response = in.readLine();
                System.out.println("Ответ сервера: " + response);
            }
        } catch (UnknownHostException e) {
            System.err.println("Неизвестный хост: " + HOST);
        } catch (IOException e) {
            System.err.println("Ошибка ввода-вывода: " + e.getMessage());
        }
    }
}
