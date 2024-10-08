**Requirements**

1、yii2

2、php >= 7.4

3、RdKafka扩展，扩展安装如下 
````text
    通过git源码编译安装：

    安装librdkafka：
    git clone https://github.com/edenhill/librdkafka.git
    cd librdkafka
    ./configure
    make && make install
    
    安装php-rdkafka扩展：
    git clone https://github.com/arnaud-lb/php-rdkafka.git
    cd php-rdkafka
    phpize
    ./configure --with-php-config=/usr/local/php7.4/bin/php-config
    make && make install

    ubuntu系统环境下安装：

    sudo apt-get install librdkafka-dev
    pecl install rdkafka
    
    php.ini配置：
    extension = rdkafka.so
````
kafka服务使用docker镜像文件构建，示例：tmp-docker-compose.yml

**yii2使用kafka**

````text
 composer require dnkfk/yii2-kafka
 ````

**消费者组件配置**

component/kafka.php
````php
    return [
        'class'   => \Dnkfk\KafkaConnection::class,
        'conn'    => ['localhost:9092'], //连接主机配置
        'topics'  => ['test_topic'], //所有需要订阅的主题
        'binding' => [
            [
                'consumer' => 'test_consumer', //消费者名称
                'group'    => 'test_group', //消费组
                'topics'   => ['test_topic'], //订阅主题
                'callback' => TestConsumer::class, //消费者回调
            ]
        ]
    ];
````
**发布消息到主题代码示例**

直接发布消息
````php
    $msg = ['msg' => 'test']; //$msg可以是数组或字符串
    \Yii::$app->kafka->produce($msg, 'test_topic');
````
批量发布消息

````php
    //构建生产者主题
    $producer = \Yii::$app->kafka->getProducer('test_topic');
    //批量发布消息到主题
    for ($i = 0; $i < 10; $i++) {
        $producer->produceMsg("message {$i}");
    }
    //等待消息全部发布完成
    $producer->wait();
````

**启动消费者**

````text
php yii kafka/consume test_consumer
````

**消费者消费代码示例**
````php
    use Dnkfk\ConsumerInterface;
    use Dnkfk\Message;
    
    class TestConsumer implements ConsumerInterface
    {
    
        /**
         * 执行消费
         *
         * @param Message $message
         * @return void
         */
        public function execute(Message $message)
        {
            $payload = json_decode($message->getPayload(), true);
    
            var_dump($payload); //输出：['msg' => 'test']
        }
    }
````
