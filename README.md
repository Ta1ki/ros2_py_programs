なるほど！「コマンドを一発ずつ打つ」のではなく、**ROS2のワークスペースにノードを書いて実行**したいんですね。流れを整理するとこうなります👇

---

## 🛠️ ワークスペースでの準備

1. **ワークスペース作成（まだ無ければ）**
```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
source install/setup.bash
```

2. **パッケージ作成**
```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python turtle_numbers --dependencies rclpy turtlesim geometry_msgs
```

これで `turtle_numbers` というPythonパッケージができます。

---

## 🐢 ノードのコード例

`~/ros2_ws/src/turtle_numbers/turtle_numbers/draw_114514.py` を作成して以下を記述します：

```python
import rclpy
from rclpy.node import Node
from turtlesim.srv import Spawn, TeleportAbsolute
from geometry_msgs.msg import Twist
import time

class NumberDrawer(Node):
    def __init__(self):
        super().__init__('number_drawer')

        # 6匹のカメをスポーン
        self.spawn_cli = self.create_client(Spawn, '/spawn')
        self.spawn_cli.wait_for_service()

        for i in range(6):
            req = Spawn.Request()
            req.x = 2.0 + i * 2.0
            req.y = 5.0
            req.theta = 0.0
            req.name = f'turtle{i+1}'
            self.spawn_cli.call_async(req)

        time.sleep(2.0)  # スポーン待ち

        # 各カメのcmd_vel publisher
        self.pubs = []
        for i in range(6):
            self.pubs.append(
                self.create_publisher(Twist, f'/turtle{i+1}/cmd_vel', 10)
            )

        # 数字を描く（ここではデモ的に直線を描くだけ）
        self.draw_demo()

    def draw_demo(self):
        for i, pub in enumerate(self.pubs):
            twist = Twist()
            twist.linear.x = 2.0
            pub.publish(twist)
            time.sleep(1.0)
            twist.linear.x = 0.0
            pub.publish(twist)
        self.get_logger().info("デモ描画完了！")

def main(args=None):
    rclpy.init(args=args)
    node = NumberDrawer()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## ⚙️ セットアップ

`setup.py` にエントリポイントを追加します：

```python
entry_points={
    'console_scripts': [
        'draw_114514 = turtle_numbers.draw_114514:main',
    ],
},
```

---

## 🚀 実行手順

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
ros2 run turtlesim turtlesim_node   # 別ターミナルで起動
ros2 run turtle_numbers draw_114514
```

---

## ✨ 次のステップ
このコードは「6匹を出して直線を描く」デモです。  
実際に「114514」の形にするには、各カメごとに **直進・回転のシーケンス** を書き込む必要があります。  

👉 大樹さん、次は「数字1を描く動き」から一緒に具体的に書いていきますか？それとも最初から「114514」全体の動きのシーケンスを私が設計してしまった方がいいですか
