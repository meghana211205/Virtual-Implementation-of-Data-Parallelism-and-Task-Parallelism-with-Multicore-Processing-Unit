# Virtual Implementation of Data Parallelism and Task Parallelism with Multicore Processing Unit

## Project Description

This project demonstrates how modern multicore processors execute multiple tasks simultaneously using parallel programming techniques.

Two parallel models are implemented:

- **Data Parallelism**
- **Task Parallelism**

The project is implemented in Java using the Concurrency Framework, including:

- `ExecutorService`
- Thread Pools
- `Future` Interface

The goal is to show how parallel computing improves performance and responsiveness compared to sequential execution.

## Technologies Used

- Java
- Multithreading
- ExecutorService
- Java Concurrency API
- GitHub

## Background: What is Parallelism?

A **sequential** program runs one instruction after another on a single core, so a large job finishes only as fast as that one core allows. A modern processor has several cores, and **parallel** programming splits the work so those cores run at the same time, finishing sooner.

This project shows the two classic ways to use multiple cores, which differ in *what* gets split up.

### Data Parallelism vs Task Parallelism

| | Data Parallelism | Task Parallelism |
|---|---|---|
| What is split | one big **dataset**, into chunks | one program, into different **tasks** |
| Each thread does | the **same** operation on a different chunk | a **different** operation |
| Example here | summing parts of an array | downloading a file while playing music |
| Analogy | ten people each counting one box of the same stock | one person cooks while another lays the table |

In short: **data parallelism** runs the *same work on different data*, while **task parallelism** runs *different work at the same time*.

## How It Works

Both programs use Java's `ExecutorService`, a thread pool that manages a fixed number of worker threads for you, so you do not create or track threads by hand.

1. `Executors.newFixedThreadPool(n)` creates a pool of `n` worker threads.
2. Work is handed to the pool with `submit(...)`, which returns a `Future` (a placeholder for a result that is still being computed).
3. For the array sum, each thread returns a partial total; `future.get()` waits for each result, and the partial totals are added into the final answer.
4. `shutdown()` tells the pool to stop accepting new work and finish what it has, and `awaitTermination(...)` waits for the tasks to complete.

The key idea: instead of one core doing all the work in order, several cores do pieces of it at the same time, and the results are combined at the end.

## Project Structure

```
OS-Parallelism-Project
│
├── DataParallelismSum.java
├── TaskParallelismMusicDemo.java
└── README.md
```

## Data Parallelism

Data Parallelism divides a large dataset into smaller segments. Each segment is processed simultaneously by separate threads. The partial results from each thread are then combined to produce the final output.

**Example in this project:** array summation using multiple threads.

### Data Parallelism Code

```java
import java.util.concurrent.*;

class DataParallelismSum {

    public static int parallelSum(int[] data, int numThreads) throws InterruptedException, ExecutionException {

        ExecutorService executor = Executors.newFixedThreadPool(numThreads);

        int chunkSize = data.length / numThreads;
        Future<Integer>[] results = new Future[numThreads];

        for (int i = 0; i < numThreads; i++) {

            int start = i * chunkSize;
            int end = (i == numThreads - 1) ? data.length : start + chunkSize;

            final int s = start, e = end;

            results[i] = executor.submit(() -> {
                int sum = 0;
                for (int j = s; j < e; j++)
                    sum += data[j];
                return sum;
            });
        }

        int total = 0;

        for (Future<Integer> fut : results)
            total += fut.get();

        executor.shutdown();

        return total;
    }

    public static void main(String[] args) throws Exception {

        int[] data = {1,2,3,4,5,6,7,8};

        int numThreads = 3;

        int result = parallelSum(data, numThreads);

        System.out.println("Parallel Sum: " + result);
    }
}
```

## Task Parallelism

Task Parallelism executes multiple independent tasks simultaneously. Each thread performs a different operation.

**Example in this project:**

- Downloading a file from the internet
- Playing a music file

Both operations run concurrently.

### Task Parallelism Code

```java
import java.io.*;
import java.net.*;
import java.util.concurrent.*;

public class TaskParallelismMusicDemo {

    static class DriveOperations {

        public void downloadFile(String fileUrl, String outputPath) {

            try {

                URL url = new URL(fileUrl);
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();

                InputStream in = conn.getInputStream();
                FileOutputStream fos = new FileOutputStream(outputPath);

                byte[] buffer = new byte[8192];
                int bytesRead;

                while ((bytesRead = in.read(buffer)) != -1) {
                    fos.write(buffer,0,bytesRead);
                }

                fos.close();
                in.close();

                System.out.println("Download Completed");

            } catch(Exception e){
                e.printStackTrace();
            }
        }
    }

    public static void playMusic(String musicPath){

        try{

            File soundFile = new File(musicPath);

            if(soundFile.exists()){
                java.awt.Desktop.getDesktop().open(soundFile);
                System.out.println("Music Playing...");
            }

        }catch(Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {

        ExecutorService executor = Executors.newFixedThreadPool(2);

        DriveOperations driveOps = new DriveOperations();

        Runnable downloadTask =
                () -> driveOps.downloadFile("imageURL","image.jpg");

        Runnable musicTask =
                () -> playMusic("music.mpeg");

        executor.submit(downloadTask);
        executor.submit(musicTask);

        executor.shutdown();

        executor.awaitTermination(5,TimeUnit.MINUTES);

        System.out.println("All Tasks Completed");
    }
}
```

> **Note:** In `TaskParallelismMusicDemo`, replace `"imageURL"` with a real file URL
> (for example `"https://example.com/image.jpg"`) and `"music.mpeg"` with the path to a
> real audio file on your computer for the demo to download and play actual files.

## How to Run

Compile the programs:

```bash
javac DataParallelismSum.java
javac TaskParallelismMusicDemo.java
```

Run the programs:

```bash
java DataParallelismSum
java TaskParallelismMusicDemo
```

## Expected Output

**Data Parallelism**

```
Parallel Sum: 36
```

**Task Parallelism**

```
Download Completed
Music Playing...
All Tasks Completed
```

## Key Takeaways

- Multicore processors can run several threads at the same time, reducing total run time.
- **Data parallelism** suits large datasets where the same calculation is repeated (summing, searching, image processing).
- **Task parallelism** suits programs that must do several unrelated jobs at once (downloading while playing, handling many users).
- Java's `ExecutorService` and `Future` make parallelism easier and safer than managing raw threads.

## License

This project is released under the MIT License. You are free to use, copy, modify, and share it, provided the original copyright notice is kept. It is provided "as is", without warranty of any kind.
