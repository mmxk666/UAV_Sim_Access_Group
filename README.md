# UAV_Sim_Access_Group
import time
import random

# --- 组员B (张渴瑞)：Packet 数据结构 ---
class Packet:
    def __init__(self, packet_id, src_id, dst_id, size, gen_time):
        self.packet_id = packet_id
        self.src_id = src_id
        self.dst_id = dst_id
        self.size = size
        self.gen_time = gen_time

    def __repr__(self):
        return f"Packet(ID:{self.packet_id}, {self.src_id}->{self.dst_id})"

# --- 组员B (张渴瑞)：Traffic 业务发生器 ---
class TrafficGenerator:
    def __init__(self, interval_sec=1.0):
        self.interval = interval_sec
        self.packet_counter = 0
        self.last_gen_time = -1.0

    def generate(self, current_time):
        if current_time - self.last_gen_time >= self.interval:
            self.packet_counter += 1
            self.last_gen_time = current_time
            return Packet(self.packet_counter, 1, 2, 1000, current_time)
        return None

# --- 组员C (罗巧巧)：Queue 队列管理器 ---
class QueueManager:
    def __init__(self, max_size=5):
        self.queue = []
        self.max_size = max_size
        self.drop_count = 0

    def enqueue(self, packet):
        if len(self.queue) < self.max_size:
            self.queue.append(packet)
            return True
        else:
            self.drop_count += 1
            return False

    def dequeue(self):
        if len(self.queue) > 0:
            return self.queue.pop(0)
        return None

    def is_empty(self):
        return len(self.queue) == 0

# --- 组员D (查俊杰)：Channel 基础信道 Mock (本周升级版) ---
class MockChannel:
    def send_packet(self, packet):
        # 模拟传输延迟 0.1 秒
        time.sleep(0.1) 
        # 模拟 10% 的随机丢包率
        if random.random() < 0.1:
            return False
        return True

# --- 组员E (杨津瑞)：Logger 过程记录 ---
class SimLogger:
    def log_event(self, module_name, event_msg):
        current_time = round(time.time() % 100, 2)
        # 修复了 f-string 格式
        print(f"[{current_time:05.2f}s] [{module_name:^7}] {event_msg}")

# --- 组员A (金灵)：MAC 预留接口 ---
class MACBase:
    def check_channel(self):
        pass
    def handle_collision(self):
        pass

# --- 主程序：纵向串联 ---
if __name__ == "__main__":
    logger = SimLogger()
    logger.log_event("System", "=== 接入组基础模块初始化成功 ===")

    traffic = TrafficGenerator(interval_sec=0.5)
    queue = QueueManager(max_size=3) # 故意设小一点，方便测试丢包
    channel = MockChannel()
    mac = MACBase()

    logger.log_event("System", "开始模拟单跳发送流程...")

    # 模拟 5 秒的时间流逝
    for step in range(50):
        t = step * 0.1  # 每次步进 0.1 秒
        
        new_packet = traffic.generate(t)
        if new_packet:
            logger.log_event("Traffic", f"生成了新包裹 -> {new_packet}")
            success = queue.enqueue(new_packet)
            if success:
                logger.log_event("Queue", f"包裹 {new_packet.packet_id} 入队 (队列: {len(queue.queue)}/3)")
            else:
                logger.log_event("Queue", f"!!! 队列已满, 丢弃包裹 {new_packet.packet_id} !!!")

        if not queue.is_empty():
            pkt_to_send = queue.dequeue()
            logger.log_event("MAC", f"调度发送包裹 {pkt_to_send.packet_id}")
            tx_result = channel.send_packet(pkt_to_send)
            if tx_result:
                logger.log_event("Channel", f"包裹 {pkt_to_send.packet_id} 发送成功")
            else:
                logger.log_event("Channel", f"包裹 {pkt_to_send.packet_id} 发送失败 (模拟丢包)")

    logger.log_event("System", f"=== 模拟结束. 共丢弃 {queue.drop_count} 个包裹 ===")
