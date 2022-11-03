# 入门

https://zhuanlan.zhihu.com/p/110521147

https://siddhi.io/en/v5.1/docs/siddhi-as-a-java-library/ 

```java
package org.github.demo;

import io.siddhi.core.SiddhiAppRuntime;
import io.siddhi.core.SiddhiManager;
import io.siddhi.core.event.Event;
import io.siddhi.core.stream.input.InputHandler;
import io.siddhi.core.stream.output.StreamCallback;
import io.siddhi.core.util.EventPrinter;


public class sihhidhDemo {

    public static void main(String[] args) throws InterruptedException {
        // Create Siddhi Manager
        SiddhiManager siddhiManager = new SiddhiManager();

        //Siddhi Application
        String siddhiApp = "" +
                "define stream StockStream (symbol string, price float, volume long); " +
                "" +
                "@info(name = 'query1') " +
                "from StockStream[volume < 150] " +
                "select symbol, price " +
                "insert into OutputStream;";

        //Generate runtime
        SiddhiAppRuntime siddhiAppRuntime = siddhiManager.createSiddhiAppRuntime(siddhiApp);

        //Adding callback to retrieve output events from stream
        siddhiAppRuntime.addCallback("OutputStream", new StreamCallback() {
            @Override
            public void receive(Event[] events) {
                /*EventPrinter.print(events);
                //To convert and print event as a map
                EventPrinter.print(toMap(events));*/
                int n=0;
                int len=events.length;
                while (n < len) {
                    System.out.println(events[n]);
                    n++;
                }
            }
        });

        //Get InputHandler to push events into Siddhi
        InputHandler inputHandler = siddhiAppRuntime.getInputHandler("StockStream");

        //Start processing
        siddhiAppRuntime.start();

        //Sending events to Siddhi
        int count = 0;
        while (count < 100) {
            inputHandler.send(new Object[]{"IBM", 700f, 100L});
            inputHandler.send(new Object[]{"WSO2", 60.5f, 200L});
            inputHandler.send(new Object[]{"GOOG", 50f, 30L});
            inputHandler.send(new Object[]{"IBM", 76.6f, 400L});
            inputHandler.send(new Object[]{"WSO2", 45.6f, 50L});
            count ++;
            Thread.sleep(3000);
        }

        //Shutdown runtime
        siddhiAppRuntime.shutdown();

        //Shutdown Siddhi Manager
        siddhiManager.shutdown();

    }

}
```

