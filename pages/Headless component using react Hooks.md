- Hook:
	- ``` jsx
	      import { useState, useEffect, useRef } from "react";
	      import { calculateTimeLeft } from "../utils";
	      
	      const useCountdown = date => {
	        const initialTimeLeft = calculateTimeLeft(date);
	        const [timeLeft, setTimeLeft] = useState(initialTimeLeft);
	        const timer = useRef();
	      
	        useEffect(() => {
	          timer.current = setInterval(() => {
	            setTimeLeft(calculateTimeLeft(date));
	          }, 1000);
	      
	          return () => {
	            if (timer.current !== undefined) {
	              clearInterval(timer.current);
	            }
	          };
	        }, [date]);
	      
	        let isValidDate = true,
	          isValidFutureDate = true;
	      
	        if (timeLeft === null) isValidDate = false;
	        if (timeLeft && timeLeft.seconds === undefined) isValidFutureDate = false;
	      
	        return { isValidDate, isValidFutureDate, timeLeft };
	      };
	      
	      export default useCountdown;
	  ```
- Hooks 封装了全部的功能逻辑，同样可以重用组件并将逻辑与 UI 分离
	- ``` jsx
	      import useCountdown from './use-countdown'; // importing the custom hook
	      
	      function App() {
	        const date = new Date("2023-01-01"); // New year!
	        const { timeLeft, isValidDate, isValidFutureDate } = useCountdown(date);
	      
	        return (<>
	            <FirstCountdownUI 
	              timeLeft={timeLeft} 
	              isValidDate={isValidDate} 
	              isValidFutureDate={isValidFutureDate} 
	            />
	            
	            <SecondCountdownUI 
	              timeLeft={timeLeft} 
	              isValidDate={isValidDate} 
	              isValidFutureDate={isValidFutureDate} 
	             />
	        </>);
	      }
	      
	      export default App;
	  ```
-