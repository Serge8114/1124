# 1124
import java.io.*;
import java.net.*;

public class EchoServer {
    private ServerSocket serverSocket;

    public EchoServer(int port) throws IOException {
        serverSocket = new ServerSocket(port);
        System.out.println("Эхо-сервер запущен на порту " + port);
    }

    public void start() {
        while (true) {
            try {
                Socket clientSocket = serverSocket.accept();
                System.out.println("Клиент подключен: " + clientSocket.getInetAddress());
                new Thread(() -> handleClient(clientSocket)).start();
            } catch (IOException e) {
                System.err.println("Ошибка при принятии подключения: " + e.getMessage());
            }
        }
    }

    private void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) { // autoFlush = true

            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                System.out.println("Получено: " + inputLine);
                
                // Вариант 1: Базовый эхо — возвращаем строку дважды через пробел
                String response = inputLine + " " + inputLine;
                out.println(response);
                System.out.println("Отправлено: " + response);
            }
        } catch (IOException e) {
            System.err.println("Ошибка при обработке клиента: " + e.getMessage());
        } finally {
            try {
                clientSocket.close();
                System.out.println("Клиент отключен");
            } catch (IOException e) {
                System.err.println("Ошибка при закрытии сокета: " + e.getMessage());
            }
        }
    }

    public static void main(String[] args) {
        int port = 8080; // стандартный порт, можно изменить
        try {
            EchoServer server = new EchoServer(port);
            server.start();
        } catch (IOException e) {
            System.err.println("Не удалось запустить сервер: " + e.getMessage());
        }
    }
}
