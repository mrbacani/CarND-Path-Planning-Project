# CarND-Path-Planning-Project
This project is my solution to Udacity's path planning project for ECE 495. The task was to use a simple state machine and cost functions to determine when it is appropriate to switch lanes such that the ego travels a 2 mile distance within 3 mins. The ego must also not exceed speed, acceleration, and jerk limits and of course avoid collisions with other vehicles. 
## Solution
I first determined if there were cars within 25m ahead of the ego and 15m behind the ego. If there was a car, I calculated its distance from the ego and its difference from the threshold. This distance minus the threshold is used in the cost function in the state machine, implemented using if else statements. The state machine first determines the current state and then calculates the cost of each future state, including cost of switching and cost of staying. For the cost function, I used the percent difference of the distance between the ego and other cars and the threshold value. This can be seen in the code snippet below:

    if (d >= 0 && d < 4) {
		if (check_car_s > car_s && (check_car_s - car_s) < vehicle_ahead_threshold) {
        	left_lane = true;
			left_lane_distance += (vehicle_ahead_threshold - (check_car_s - car_s)) / vehicle_ahead_threshold
		} else if (car_s > check_car_s && (car_s - check_car_s) < vehicle_behind_threshold) {
			left_lane = true;
			left_lane_distance += (vehicle_behind_threshold - (car_s - check_car_s)) / vehicle_behind_threshold;
		}
	} else if (d >= 4 && d < 8) {
		if (check_car_s > car_s && (check_car_s - car_s) < vehicle_ahead_threshold) {
	        middle_lane = true;
			middle_lane_distance += (vehicle_ahead_threshold - (check_car_s - car_s)) / vehicle_ahead_threshold;
		} else if (car_s > check_car_s && (car_s - check_car_s) < vehicle_behind_threshold) {
			middle_lane = true;
			middle_lane_distance += (vehicle_behind_threshold - (car_s - check_car_s)) / vehicle_behind_threshold;
		}
	} else if (d >= 8 && d < 12) {
		if (check_car_s > car_s && (check_car_s - car_s) < vehicle_ahead_threshold) {
			right_lane = true;
			right_lane_distance += (vehicle_ahead_threshold - (check_car_s - car_s)) / vehicle_ahead_threshold;
	    } else if (car_s > check_car_s && (car_s - check_car_s) < vehicle_behind_threshold) {
			right_lane = true;
			right_lane_distance += (vehicle_behind_threshold - (car_s - check_car_s)) / vehicle_behind_threshold;
		}
	}
I then mapped the value to a range of 0 to 1 using the following function to come up with the cost value:
![equation](http://www.sciweavers.org/tex2img.php?eq=1-%20e%5E%7B-x%7D%20&bc=White&fc=Black&im=jpg&fs=12&ff=arev&edit=0)
The percent difference value is never below 0 so the cost function would never return a negative value. I then compared which future state had the lowest cost and made changes according to the result. This can be seen in the code snippet below:

    					
	// Using cost functions, check if it is worth it to switch
	if (lane == 0 && left_lane) {
		double cost_switch = switch_lane*(1 - exp(-middle_lane_distance));
		double cost_stay = stay_lane*(1 - exp(-left_lane_distance));
		too_close = true;

		std::cout << "Cost to stay: " << std::fixed << cost_stay << std::endl;
		std::cout << "Cost to switch to middle: " << std::fixed << cost_switch << std::endl;
		if (cost_switch < cost_stay) {
			lane = 1;
		} 
	} else if (lane == 1 && middle_lane) {
		double cost_switch_left = switch_lane*(1 - exp(-left_lane_distance));
		double cost_switch_right = switch_lane*(1 - exp(-right_lane_distance));
		double cost_stay = stay_lane*(1 - exp(-middle_lane_distance));
		too_close = true;
		
		std::cout << "Cost to stay: " << std::fixed << cost_stay << std::endl;
		std::cout << "Cost to switch left: " << std::fixed << cost_switch_left << std::endl;
		std::cout << "Cost to switch right: " << std::fixed << cost_switch_right << std::endl;
		if (cost_switch_left < cost_switch_right && cost_switch_left < cost_stay) {
			lane = 0;
		} else if (cost_switch_right < cost_switch_left && cost_switch_right < cost_stay) {
			lane = 2;
		} else if (cost_switch_right == cost_switch_left && cost_switch_right < cost_stay) {
			lane = 2;						
		}
	} else if (lane == 2 && right_lane) {
		double cost_switch = switch_lane*(1 - exp(-middle_lane_distance));
		double cost_stay = stay_lane*(1 - exp(-right_lane_distance));
		too_close = true;

		std::cout << "Cost to stay: " << std::fixed << cost_stay << std::endl;
		std::cout << "Cost to switch to middle: " << std::fixed << cost_switch << std::endl;
		if (cost_switch < cost_stay) {
			lane = 1;
		} 
	}