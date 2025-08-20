import rclpy
from rclpy.node import Node
from turtlesim.srv import Spawn
from geometry_msgs.msg import Twist
import math
import time

class TurtleDrawer(Node):
    def __init__(self):
        super().__init__('turtle_drawer')
        self.turtle_names = ['turtle2', 'turtle3', 'turtle4', 'turtle5', 'turtle6', 'turtle7']
        self.turtle_publishers = {}
        self.spawn_positions = self.calculate_spawn_positions()
        self.spawn_orientations = self.calculate_spawn_orientations()
        self.spawn_turtles()
        self.create_publishers()
        self.draw_digits()

    def calculate_spawn_positions(self):
        x_positions = [2.0, 3.5, 5.0, 6.0, 8.0, 9.5]
        y_positions = [8.0, 8.0, 4.0, 5.0, 6.5, 4.0]
        return [(x, y) for x, y in zip(x_positions, y_positions)]

    def calculate_spawn_orientations(self):
        return [
            math.radians(270),
            math.radians(270),
            math.radians(90),
            math.radians(0),
            math.radians(270),
            math.radians(90)
        ]

    def spawn_turtles(self):
        client = self.create_client(Spawn, 'spawn')
        while not client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Waiting for spawn service...')
        for i, name in enumerate(self.turtle_names):
            x, y = self.spawn_positions[i]
            theta = self.spawn_orientations[i]
            request = Spawn.Request()
            request.x = x
            request.y = y
            request.theta = theta
            request.name = name
            future = client.call_async(request)
            rclpy.spin_until_future_complete(self, future)
self.get_logger().info(f'Spawned {name} at ({x}, {y}) with theta {theta}')

    def create_publishers(self):
        for name in self.turtle_names:
            topic = f'/{name}/cmd_vel'
            self.turtle_publishers[name] = self.create_publisher(Twist, topic, 10)

    def move_turtle(self, name, linear, angular, duration):
        twist = Twist()
        twist.linear.x = linear
        twist.angular.z = angular
        end_time = time.time() + duration
        while time.time() < end_time:
            self.turtle_publishers[name].publish(twist)
            time.sleep(0.1)
        twist.linear.x = 0.0
        twist.angular.z = 0.0
        self.turtle_publishers[name].publish(twist)

    def draw_digits(self):
        # 1
        self.move_turtle('turtle2', linear=1.0, angular=0.0, duration=2.0)
        self.move_turtle('turtle3', linear=1.0, angular=0.0, duration=2.0)

        # 4
        self.move_turtle('turtle4', linear=1.0, angular=0.0, duration=2.0)
        self.move_turtle('turtle4', linear=0.0, angular=math.radians(120), duration=1.0)
        self.move_turtle('turtle4', linear=1.0, angular=0.0, duration=1.0)
        self.move_turtle('turtle4', linear=0.0, angular=math.radians(150), duration=1.0)
        self.move_turtle('turtle4', linear=1.0, angular=0.0, duration=2.0)

        # 5
        self.move_turtle('turtle5', linear=1.0, angular=1.0, duration=math.pi)
        self.move_turtle('turtle5', linear=0.0, angular=math.radians(-90), duration=1.0)
        self.move_turtle('turtle5', linear=1.0, angular=0.0, duration=1.0)
        self.move_turtle('turtle5', linear=0.0, angular=math.radians(-90), duration=1.0)
        self.move_turtle('turtle5', linear=1.0, angular=0.0, duration=1.5)

        # 1
        self.move_turtle('turtle6', linear=1.0, angular=0.0, duration=2.0)

        # 4
        self.move_turtle('turtle7', linear=1.0, angular=0.0, duration=2.0)
        self.move_turtle('turtle7', linear=0.0, angular=math.radians(120), duration=1.0)
        self.move_turtle('turtle7', linear=1.0, angular=0.0, duration=1.0)
        self.move_turtle('turtle7', linear=0.0, angular=math.radians(150), duration=1.0)
        self.move_turtle('turtle7', linear=1.0, angular=0.0, duration=2.0)

    def main(args=None):
        rclpy.init(args=args)
        node = TurtleDrawer()
        rclpy.spin(node)
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
