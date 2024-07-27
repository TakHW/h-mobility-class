``` python 3
    def timer_callback(self):

        if self.lidar_data is not None and self.lidar_data.data is True:
            # 라이다가 장애물을 감지한 경우
            self.steering_command = 0 
            self.left_speed_command = 0 
            self.right_speed_command = 0 

        elif self.traffic_light_data is not None and self.traffic_light_data.data == 'Red':
            # 빨간색 신호등을 감지한 경우
            for detection in self.detection_data.detections:
                if detection.class_name=='traffic_light':
                    x_min = int(detection.bbox.center.position.x - detection.bbox.size.x / 2) # bbox의 좌측상단 꼭짓점 x좌표
                    x_max = int(detection.bbox.center.position.x + detection.bbox.size.x / 2) # bbox의 우측하단 꼭짓점 x좌표
                    y_min = int(detection.bbox.center.position.y - detection.bbox.size.y / 2) # bbox의 좌측상단 꼭짓점 y좌표
                    y_max = int(detection.bbox.center.position.y + detection.bbox.size.y / 2) # bbox의 우측하단 꼭짓점 y좌표

                    if y_max < 150:
                        # 신호등 위치에 따른 정지명령 결정
                        self.steering_command = 0 
                        self.left_speed_command = 0 
                        self.right_speed_command = 0
        else:
            if self.lane_data is None:
                self.steering_command = 0
            else:    
                target_point = (self.lane_data.target_x, self.lane_data.target_y) # 차선의 중심점
                car_center_point = (320, 179) # roi가 잘린 후 차량 앞 범퍼 중앙 위치

                target_slope = DMFL.calculate_slope_between_points(target_point, car_center_point)
                
                if target_slope > 0:
                    self.steering_command =  7 # 예시 속도 값 (7이 최대 조향) 
                elif target_slope < 0:
                    self.steering_command =  -7
                else:
                    self.steering_command = 0


                #### 이전 예시 ####
                # if self.lane_data.slope > 0:
                #     self.steering_command =  7 # 예시 속도 값 (7이 최대 조향) 
                # elif self.lane_data.slope < 0:
                #     self.steering_command =  -7
                # else:
                #     self.steering_command = 0
                ###########################
            self.left_speed_command = 100  # 예시 속도 값 (255가 최대 속도)
            self.right_speed_command = 100  # 예시 속도 값 (255가 최대 속도)

        # 모션 명령 메시지 생성 및 퍼블리시
        motion_command_msg = MotionCommand()
        motion_command_msg.steering = self.steering_command
        motion_command_msg.left_speed = self.left_speed_command
        motion_command_msg.right_speed = self.right_speed_command
        self.publisher.publish(motion_command_msg)

```




최종 알고리즘
![[Pasted image 20240726200831.png]]
