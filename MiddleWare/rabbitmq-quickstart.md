rabbitmqctl add_user jiuyou2020 123456
rabbitmqctl set_user_tags jiuyou2020 administrator
rabbitmqctl set_permissions -p / jiuyou2020 ".*" ".*" ".*"