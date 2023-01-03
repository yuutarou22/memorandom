# イチから学ぶ、ThreadとExecutorと、Coroutineと

## Thread(スレッド)とは？

プログラムの処理のまとまりのこと。

- **シングルスレッド**
  - 処理が１列に処理されていくこと
- **マルチスレッド**
  - 処理が２列以上あり、並行して同時進行で処理されていくこと

## Threadを使う目的とは？

- **異なることを同時に実行する**
  - ネットワーク通信しつつ、画面操作を可能にする
- **処理時間を短くする**
  - 大きな処理を分割し、小さな処理となったものをスレッドで並列実行すると短縮できる
- **単位時間（sec,msec）あたりの処理量を増やす**
  - コンピュータの処理性能によるが、複数のスレッドを実行することが可能であれば、単位時間あたりの処理量を増やすことが可能

## Threadを使うには？

主に以下の2つの方法がある。

- ①Threadクラスを継承したクラスを作成し、インスタンス化して使う
- ②Runnableインタフェースを実装したクラスを作成し、それをThreadクラスで実行して使う

### ①Threadクラスを継承したクラス

※以降、スレッドの本体：**ThreadSample**, スレッドの呼び出し元：**ThreadExecutor**とする

```java
class ThreadSample extends Thread {
    public void run() {
        System.out.println("Thread test");
    }
}
```

この『Threadクラスを継承したクラス』のインスタンスを生成し実行する。

```java
class ThreadExecutor {
    public static void main(String[] args) {
        ThreadSample t = new ThreadSample();
        t.start(); // run()ではなく、start()で実行される

        /* 複数呼び出す場合 */
        ThreadSample t1 = new ThreadSample();
        ThreadSample t2 = new ThreadSample();
        ThreadSample t3 = new ThreadSample();
        t1.start();
        t2.start();
        t3.start();
    }
}
```

![スレッドのイメージ](https://www.bold.ne.jp/engineer-club/wp-content/uploads/2020/01/%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E3%81%AF%E4%B8%A6%E5%88%97%E3%81%A7%E5%8B%95%E3%81%8F.png)

### ②Runnableインタフェースを実装したクラス

```java
class RunnableSample implements Runnable {
    public void run() {
        // 実行中のスレッドの名前を取得する（この時点で実体はまだ無いため）
        String threadName = Thread.currentThread().getName();
        System.out.println("スレッド" + threadName + "で動作中");
    }
}
```

```java
class ThreadExecutor {
    public static void main(String[] args) {
        RunnableSample r = new RunnableSample();
        Thread t = new Thread(r); // RunnableインスタンスをThreadへ受け渡し、インスタンス化
        t.start(); // Runnableのrun()が実行される

        /* 複数呼び出す場合 */
        Thread t1 = new Thread(r);
        Thread t2 = new Thread(r);
        Thread t3 = new Thread(r);
        t1.start();
        t2.start();
        t3.start();
    }
}
```

![Runnableイメージ](https://www.bold.ne.jp/engineer-club/wp-content/uploads/2020/01/%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E3%81%8CRunnable%E3%82%92%E5%91%BC%E3%81%B3%E5%87%BA%E3%81%99.png)

## Threadでよく使われるメソッドとは？

- **getId/getName**
  - JVMが割り当てた『スレッド識別子』を取得可能
- **currentThread**
  - Runnableで実装した場合、まだJVMにスレッドは割り当てられていないので、getId/getNameが呼び出せない。そこで現在の自分自身のスレッド確認->識別子の順に見る必要がある。
- **sleep**
  - スレッドの処理を一時停止する。
- **join**
  - 特定のスレッドの処理が終わるのを待つ場合に使用する。
- **interrupt**
  - 処理中のスレッドへ中断を伝える（割り込み）際に使用する。

### Thread.join

```java
class ThreadSample extends Thread {
    public void run() {
        System.out.println("sleep START");
        tyr {
            Thread.sleep(10000L);
        } catch (InterruptedException e) {}
        System.out.println("sleep END");
    }
}
```

```java
class ThreadExecutor {
    public static void main(String[] args) {
        Thread t = new ThreadSample();
        t.start();
        System.out.println("join START");
        try {
            t.join(); // 終了を待ちたいスレッドのjoinを呼び出す
        } catch (InterruptedException e) {}

        System.out.println("join END");
    }
}
```

```
// joinあり
join START
sleep START
sleep END
join END

// joinなし
join START
sleep START
join END
sleep END
```

![joinのイメージ](https://www.bold.ne.jp/engineer-club/wp-content/uploads/2020/01/%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E3%81%AE%E7%B5%82%E4%BA%86%E3%82%92join%E3%81%A7%E5%BE%85%E3%81%A4.png)

### Thread.interrupt

- **注意（Interruptを検知できる状態）**
  - Object.wait
  - Thread.join
  - Thread.sleep
  - これ以外の時に割り込みを検知するには、`Thread.interrupted`や`Thread.isInterrupted`を呼び出す

```java
class ThreadSample extends Thread {
    public void run() {
        System.out.println("sleep START");
        tyr {
            Thread.sleep(10000L);
            System.out.println("sleep END");
        } catch (InterruptedException e) {
            System.out.println("割り込み発生！");
        }
    }
}
```

```java
class ThreadExecutor {
    public static void main(String[] args) {
        Thread t = new ThreadSample();
        t.start();

        try {
            Thread.sleep(1000L); // 1秒待機
        } catch (InterruptedException e) {}

        System.out.println("スレッド割り込みーSTART");
        t.interrupt(); // 割り込みたいスレッドのinterruptを呼び出す
        System.out.println("スレッド割り込みーEND");
    }
}
```

```
sleep START
スレッド割り込みーSTART
スレッド割り込みーEND
割り込み発生！
```

![interruptのイメージ](https://www.bold.ne.jp/engineer-club/wp-content/uploads/2020/01/%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E3%81%B8interrupt%E3%81%A7%E5%89%B2%E3%82%8A%E8%BE%BC%E3%82%80.png)

## Threadでの処理結果の受け取り方とは？

スレッドを使って並列処理を実行する場合は、大抵ネットワーク通信を行い、外部から情報を取得するケースが多い。

その処理結果を、呼び出し元は受け取りたいが、`Thread.run`と`Runnable.run`の戻り値は`void`である。

- [Thread.run](https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html#run())
- [Runnable.run](https://docs.oracle.com/javase/7/docs/api/java/lang/Runnable.html#run())

受け取る方法としては、以下の3つがある。

- **ポーリング(polling)**
  - Thread/Runnableに処理が終了したか問い合わせ、終了していたら結果を取得するメソッドを実行する
  - 郵便局員のイメージ。ポストにハガキがあれば送り届ける。なければスルー。
  - 自宅でいえば、ポストをよく見にいく子供のイメージ。年賀はがきとか。
  - https://itmanabi.com/polling/
- **コールバック(callback)**
  - Thread/Runnableへ「処理が終わった後に呼び出すオブジェクト」を渡し、終わったら所定のメソッドを呼び出してもらう
  - コール（呼び出して）バック（戻す）
  - https://wa3.i-3-i.info/word12295.html
- **データ共有用オブジェクト**
  - 処理結果を格納するオブジェクトを、呼び出し元と呼び出し先で共有する

### ポーリング

ポーリングは、そそっかしい人のイメージ。「まだかな？まだかな？」と定期的に監視している。子の帰りを待つ母親。初めてのAmazon購入品が届くのを楽しみにしている子ども。

定期的に問い合わせる処理を実装する。

```java
class ThreadSample extends Thread {
    private int result;
    private boolean finished;

    public void run() {
        for (int i = 0; i <= 10; i++) {
            result += i;
        }

        finished = true;
    }

    int getResult() {
        return result;
    }

    boolean getFinished() {
        return finished
    }
}


```

```java
class ThreadExecutor {
    public static void main(String[] args) {
        Thread t = new ThreadSample();
        t.start();


        while (true) { // 1秒ごとに終了フラグを確認しにいく
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {}

            if (t.getFinished) {
                break;
            }
        }

        int result = t.getResult();
        System.out.println("処理結果: " + result);
    }
}
```

- Thread.sleepの間に他の処理が実行できる
- 反面、定期的に処理終了を確認しにいかなければならない。

### コールバック

```java
class ThreadSample extends Thread {
    private Callback callback;

    public void run() {
        int result = 0;

        for (int i = 1; i <= 10; i++) {
            result += i;
        }

        callback.finished(result);
    }

    void setCallback(Callback callback) {
        this.callback = callback;
    }

    // run()実行後の処理がインタフェースで抽象化されている -> 汎用性が増す
    static interface Callback {
        void finished(int result);
    }
}
```

```java
class ThreadExecutor {
    public static void main(String[] args) {
        ThreadSample.Callback callback = new ThreadSample.Callback() {
            public void finished(int result) {
                System.out.println("処理結果: " + result);
            }
        };

        ThreadSample t = new ThreadSample();
        t.setCallback(callback);
        t.start();
    }
}
```

ここでは、[無名クラス（匿名クラス）](https://magazine.techacademy.jp/magazine/19027)にてThreadSample.Callbackインタフェースをインスタンス化した。

[こういう呼び出し方](https://deecode.net/?p=404)もある。

```java
public class Main {

    // インタフェースを定義
    interface AsyncFuntionCallback {
        void onAsyncFunctionFinished(boolean isSuccess);
    }

    public static void main(String[] args) {
        // 非同期処理が完了した後の処理をインスタンス化
        AsyncFunctionCallback callback = new AsyncFunctionCallback() {
            @Override
            public void onAsyncFunctionFinished(boolean isSuccess) {
                System.out.println("処理成功");
            }
        };

        // 別のメソッドに、コールバック関数を引数として渡す
        asyncFunction(callback);
    }

    private static void asyncFunction(AsyncFunctionCallback callback) {
        // 無名クラスでこのままrun()も定義
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("別スレッドで非同期処理ーSTART");
                try {
                    Thread.sleep(1000);
                    callback.onAsyncFunctionFinished(true);
                } catch(InterruptedExecprion e) {
                    callback.onAsyncFunctionFinished(false);
                }
            }
        });
        t.setDeamon(false);
        t.start();
    }
}
```

- [【補足】setDeamonについて](https://dk521123.hatenablog.com/entry/36781533)
  - プログラム終了時、スレッドが破棄される(https://docs.oracle.com/javase/jp/8/docs/api/java/lang/Thread.html#setDaemon-boolean-)
  - [デーモンスレッド](https://www.web-development-kb-ja.site/ja/java/java%E3%81%AE%E3%83%87%E3%83%BC%E3%83%A2%E3%83%B3%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E3%81%A8%E3%81%AF%E4%BD%95%E3%81%A7%E3%81%99%E3%81%8B%EF%BC%9F/968437584/)は、プログラム終了時、JVMが終了するのを妨害しないスレッドのこと。
  - [デーモン](https://wa3.i-3-i.info/word11000.html)とは、[バックグラウンド](https://wa3.i-3-i.info/word12544.html)で常に動作しているプログラムのことを指す（こともある）

### データ共有用オブジェクト

〜〜いったん省略〜〜


## 同期化・排他ロック(synchronize)

スレッドを使う際セットになるのが「処理が同時に動くことを防ぐ」という考え方である。

10個のスレッドを生成し、それぞれのスレッドで1000回+1し続けるプログラムを実行する

```java
class RunnableSample implements Runnable {
    private int count;

    public void run() {
        for (int i = 0; i<1000; i++) {
            count++;
        }
    }

    int getCount() {
        return count;
    }
}
```

```java
class ThreadExecutor {
    public static void main(String[] args) {
        // for (int i = 0; i < 100; i++) {
            RunnableSample r = new RunnableSample();
            Thread[] threads = new Thread[10];

            for (int j = 0; j < threads.length; j++) {
                // 1つのRunnableインスタンスを複数のThreadに渡している
                // Runnableはメモリ領域を共有している状態
                threads[j] = new Thread(r);
            }

            for (int j = 0; j < threads.length; j++) {
                threads[j].start();
            }

            for (int j = 0; j <threads.length; j++) {
                try {
                    threads[j].join();
                } catch (InterruptedException e) {}
            }

            System.out.println((i+1) + "回目, 結果 :" + r.getCount());
        // }
    }
}
```

変数countは[ヒープ領域](https://it-trend.jp/development_tools/article/32-0041)にあるため、+1するスレッドそれぞれでスタック領域へ持ち帰る（コピーする）ため起こる現象である。

### 同期的にThreadを実行する

synchronizedという仕組みを利用する。

**複数のThreadから、Runnableのrun()を同時に実行できないようにする**

```java
class RunnableSample implements Runnable {
    private int count;

    public synchronized void run() {
        for (int i = 0; i<1000; i++) {
            count++;
        }
    }

    int getCount() {
        return count;
    }
}
```

- KW
  - synchronizedメソッドの使用
  - synchronizedブロックの使用
  - 変数へのvolatile修飾子の指定
  - java.util.concurrent.lock.ReentrantLockの使用
  - java.util.concurrent.atomic.AtomicIntegerなどの使用


























---







## Executorとは？

https://akira-watson.com/android/asynctask.html

[Concurrent Utilities（コンカレント ユーティリティーズ）](https://docs.oracle.com/javase/jp/8/docs/api/java/util/concurrent/package-summary.html)は、並行処理を実行するためにまとめられたJava API群である。

その中でも[Executor](https://docs.oracle.com/javase/jp/8/docs/api/java/util/concurrent/Executors.html
)は、スレッドをより簡単に使えるようにするためのユーティリティクラスである。

```java
class RunnableSample implements Runnable {
    private int result;

    public void run() {
        for (int i = 1; i <= 10; i++) {
            result += i;
        }
    }

    int getResult() {
        return result;
    }
}
```

```java
class ExecutorServiceSample {
    public static void main(String[] args) {
        RunnableSample r = new RunnableSample();
        ExecutorService service = Executors.newSingleThreadExecutor();
        Future f = service.submit(r);

        try {
            f.get();
        } catch (ExecutionExcepution | InterruptedException e) {}

        service.shutdown();
        System.out.println("処理結果: " + r.getResult());
    }
}
```

- **【POINT】**
  - Runnableは変わらず使う
  - Threadの代わりにExecutorsやExecutorServiceを使う

- **【KW】**
  - [Executorsクラス](https://docs.oracle.com/javase/jp/8/docs/api/java/util/concurrent/Executors.html)
    - ここ（newSingleThreadExecutor）では単一のExecutorを管理するExecutorServiceを作成し、返している
  - [ExecutorServiceインタフェース](https://docs.oracle.com/javase/jp/8/docs/api/java/util/concurrent/ExecutorService.html) -> Executorインタフェース
    - 終了を管理するメソッド、および1つ以上の非同期タスクの進行状況を追跡する`Future`を生成できるメソッドを提供する
    - submitメソッドは、実行の取消しまたは完了の待機、あるいはその両方に使用できる`Future`を作成して返すことによって、基底メソッド`Executor.execute(Runnable)`を拡張する
  - [Future](https://docs.oracle.com/javase/jp/8/docs/api/java/util/concurrent/Future.html)
    - 1つ以上の非同期タスク（Thread）の進行状況を追跡するクラス
    - 計算が完了したかどうかのチェック、完了までの待機、結果の取得などのメソッドが用意されている。
  - [スレッドプール](https://okwave.jp/qa/q6586640.html)
    - サーバリソースは物理的に有限である。(CPU, メモリ, DISK I/O...)
    - アプリケーションからの要求の度にThreadを生成→消滅するコストは、大きな負担になる。
    - 「どうせ要求がくる」とわかっているなら、予めThreadを生成しておき、消滅させず再利用することでスレッドの生成・消滅（後処理）のコストを削減している。
    - 同時実行Threadが多すぎるのも問題なので、スレッド数を適度に抑え、キューに貯めておくこともある。

こちらもわかりやすい。

- [ExecutorServiceを使って、Javaでマルチスレッド処理](https://qiita.com/mmmm/items/f33b757119fc4dbd6aa1)
- [ラーメン屋で例えるスレッド](https://blog.websandbag.com/entry/2020/06/22/173514)
- [幸せな非同期処理ライフを満喫するための基礎から応用まで](https://qiita.com/KeithYokoma/items/4e6e9bd4e44aab63424d)

## Callable

Callableを使うと、スレッドでの実行結果をスレッドの外から直接得られる

〜〜いったん省略〜〜

## 【補足】Concurrent Utilitiesの他のクラス

- 値への操作が同期化されたAtomicInteger、AtomicBooleanなど
- 排他ロックを簡単に行うためのReentrantLock
- 同期化の処理パフォーマンスが改善されたConcurrentArrayListなど
- スレッド間での待ち合わせができるCyclicBarrier、Semaphore、Phaserなど
- 定期的な処理をスレッドで実行できるScheduledExecutorService
- スレッド実行後の後続処理を指定できるCompletableFuture
- スレッドを使って処理を効率化するためのForkJoinPool/ForkJoinTaskなど






















---

## Coroutine(コルーチン)とは？

https://qiita.com/iTakahiro/items/e39533b7fc07573a14da
https://qiita.com/kawmra/items/d024f9ab32ffe0604d39
