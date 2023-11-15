# net


#### 서버를 연결해 계산기 만들기

### 서버 프로그램 (Server.java)
이 서버 프로그램은 포트 1001에서 클라이언트의 연결을 기다리고, 연결된 클라이언트의 요청을 처리합니다.
```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;

public class Server {
    public static void main(String[] args) {
        int port = 1001;
        ExecutorService pool = Executors.newFixedThreadPool(10);

        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("서버가 포트 " + port + "에서 시작되었습니다.");

            while (true) {
                Socket clientSocket = serverSocket.accept();
                ClientHandler clientHandler = new ClientHandler(clientSocket);
                pool.execute(clientHandler);
            }
        } catch (IOException e) {
            System.err.println("서버 오류: " + e.getMessage());
            e.printStackTrace();
        }
    }
}

class ClientHandler implements Runnable {
    private Socket clientSocket;

    public ClientHandler(Socket socket) {
        this.clientSocket = socket;
    }

    @Override
    public void run() {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                try {
                    // 산술 연산 처리 로직 (간단한 구현 예시)
                    out.println("계산 결과: " + inputLine); // 여기에 적절한 연산 로직을 추가하세요.
                } catch (Exception e) {
                    out.println("오류: 잘못된 요청");
                }
            }
        } catch (IOException e) {
            System.err.println("클라이언트 핸들러 오류: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### 클라이언트 프로그램 (Client.java)
클라이언트 프로그램은 `server_info.dat` 파일에서 서버의 정보를 읽어 연결을 시도합니다. 파일이 없을 경우 기본값(`localhost`, 포트 `1234`)을 사용합니다.
```java
import java.io.*;
import java.net.*;

public class Client {
    public static void main(String[] args) {
        String serverAddress = "localhost"; // 기본 서버 주소
        int port = 1001; // 기본 포트 번호

        // server_info.dat 파일에서 서버 정보 읽기
        try (BufferedReader fileReader = new BufferedReader(new FileReader("server_info.dat"))) {
            serverAddress = fileReader.readLine();
            port = Integer.parseInt(fileReader.readLine());
        } catch (FileNotFoundException e) {
            System.out.println("서버 정보 파일을 찾을 수 없음. 기본 설정을 사용합니다.");
        } catch (IOException e) {
            System.out.println("파일 읽기 오류. 기본 설정을 사용합니다.");
        }

        // 서버에 연결
        try (Socket socket = new Socket(serverAddress, port);
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             BufferedReader stdIn = new BufferedReader(new InputStreamReader(System.in))) {

            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput); // 서버에 입력 전송
                System.out.println("서버로부터의 응답: " + in.readLine());
            }
        } catch (UnknownHostException e) {
            System.err.println("호스트 IP를 찾을 수 없습니다: " + serverAddress);
        } catch (IOException e) {
            System.err.println("서버에 연결할 수 없습니다.");
        }
    }
}
```

### 실행 방법
1. 먼저 서버 프로그램(`Server.java`)을 실행합니다.
2. 그 다음 클라이언트 프로그램(`Client.java`)을 실행합니다.
