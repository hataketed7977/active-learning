- UI Part:
	- ``` jsx
	      import React, { useState, useEffect, useRef } from "react";
	      
	      const SimpleCountdown = ({ date }) => {
	        //定义数据结构
	        const isValidDate = false, isValidFutureDate = false, timeLeft = {};
	        return (
	          <div className="countdown">
	            <h3 className="header">Simple Countdown</h3>
	            {!isValidDate && <div>Pass in a valid date props</div>}
	            {!isValidFutureDate && (
	              <div>
	                Time up, let's pass a future date to procrastinate more{" "}
	                <span role="img" aria-label="sunglass-emoji">
	                  😎
	                </span>
	              </div>
	            )}
	            {isValidDate && isValidFutureDate && (
	              <div>
	                {timeLeft.days} days, {timeLeft.hours} hours, {timeLeft.minutes}{" "}
	                minutes, {timeLeft.seconds} seconds
	              </div>
	            )}
	          </div>
	        );
	      };
	      
	      export default SimpleCountdown;
	  ```
	- ``` jsx
	      import React from "react";
	      import SimpleCountdown from "./components/simple-countdown";
	      
	      function App() {
	        const date = new Date(); 
	        return (
	          <div className="App">
	            <SimpleCountdown date={date} />
	            <hr />
	          </div>
	        );
	      }
	  ```
	-
- Logic Part: 计算数据   timeLeft，isValidDate, isValidFutureDate
	- ``` js
	      //date-util.js
	      import isValid from "date-fns/isValid";
	      
	      export const calculateTimeLeft = date => {
	        if (!isValid(date)) return null;
	        const difference = new Date(date) - new Date();
	        let timeLeft = {};
	        if (difference > 0) {
	          timeLeft = {
	            days: Math.floor(difference / (1000 * 60 * 60 * 24)),
	            hours: Math.floor((difference / (1000 * 60 * 60)) % 24),
	            minutes: Math.floor((difference / 1000 / 60) % 60),
	            seconds: Math.floor((difference / 1000) % 60)
	          };
	        }
	        return timeLeft;
	      };
	  ```
	- ``` jsx
	      const SimpleCountdown = ({ date }) => {
	        const [timeLeft, setTimeLeft] = useState(calculateTimeLeft(date));
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
	        
	        let isValidDate = true, isValidFutureDate = true;
	      
	        if (timeLeft === null) isValidDate = false;
	        if (timeLeft && timeLeft.seconds === undefined) isValidFutureDate = false
	       ....
	      }
	  ```
- 分别实现了UI和数据逻辑
	- 功能 与 UI 隔离
	- 由于 UI 在组件内部，所以任然不能重用
		- 例如：使用 SVG 和 CSS 动画创建另一个倒计时 UI
- 简单改造一下：
	- ``` jsx
	      import { useState, useEffect, useRef } from "react";
	      import { calculateTimeLeft } from "../utils";
	          
	          
	      const Countdown = ({ date, children }) => {
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
	      
	        return children({
	          isValidDate,
	          isValidFutureDate,
	          timeLeft
	        });
	      };
	      
	      export default Countdown;
	  ```
- 实现逻辑与 UI 分离，功能重用创建多个不同的 UI
	- ``` jsx
	      function App() {
	        const date = new Date("2023-01-01"); // New year!
	      
	        return (<>
	            <Countdown date={date}>
	              {(renderProps) => (
	                <FirstCountdownUI {...renderProps} />
	              )}
	            </Countdown>
	          
	            <Countdown date={date}>
	              {(renderProps) => (
	                <SecondCountdownUI {...renderProps} />
	              )}
	            </Countdown>
	      
	        </>);
	      }
	  ```