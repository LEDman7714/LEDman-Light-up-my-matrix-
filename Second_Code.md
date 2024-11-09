/### [ Отчет о работе №2 ]

Первыми были реализованы сценарии №1 и №3. На начальных этапах я делал оформление начального меню и занисался в основном Игроком. Сейчас сделан еще и сценарий №4. Создано несколько видов врагов.
Вот так выглядит код одного из противников:

    extends CharacterBody2D
    var damage : int = 4
    var en_hp : int = 8
    var radius : float = 512
    var player : Player
    var constant_vel : float = 200
    var acceleration : float = 0
    var s_frames : int = 0
    var d_timer : int = -1
    var r = 255
    var g = 255
    var b = 255
    
    # Effects
    var burn = 0
    func _ready() -> void:
    	player = get_tree().get_first_node_in_group("Player")
    	$AnimatedSprite2D.play("default")
    
    func _physics_process(delta: float) -> void:
    	var distance = (player.global_position - global_position).length()
    	if s_frames == 0:
    		if distance <= radius and en_hp > 0:
    			velocity = Vector2(constant_vel, 0).rotated(Vector2.ZERO.angle_to_point(player.global_position - global_position))
    		elif distance > radius and en_hp > 0:
    			velocity = Vector2.ZERO
    			$AnimatedSprite2D.animation = "default"
    		elif en_hp <= 0:
    			velocity = Vector2.ZERO
    			$AnimatedSprite2D.animation = "Death"
    		acceleration = 0
    	else:
    		s_frames -= 1
    		velocity *= (1 + acceleration)
    		acceleration += 0.01 * -sign(acceleration)
    	if velocity.length() > 0:
    		$AnimatedSprite2D.animation = "Walk"
    	if constant_vel < 200:
    		constant_vel += 1
    	if burn == 0:
    		if r < 255:
    			r += 1
    		if g < 255:
    			g += 1
    		if b < 255:
    			b += 1
    	modulate = Color8(r, g, b)
    	# Effects
    	if burn > 0:
    		# modulate = Color8(255, 94, 0, 255)
    		burn -= 1
    		if randi_range(1, 14) == 1:
    			en_hp -= 1
    			var dmg_number = preload("res://scenes/UI/damage_number.tscn").instantiate()
    			add_sibling(dmg_number)
    			dmg_number.global_position = global_position
    			dmg_number.text = str(1)
    			dmg_number.modulate = Color8(255, 94, 0, 255)
    	move_and_slide()
    	
    	if d_timer > 0:
    		d_timer -= 1
    	
    	for i in get_slide_collision_count():
    		var coll = get_slide_collision(i).get_collider()
    		if coll.is_in_group("Player") and s_frames == 0:
    			s_frames = 24
    			velocity = -velocity * 40
    			acceleration = -0.5
    			
    	for i in get_slide_collision_count():
    		var coll = get_slide_collision(i).get_collider()
    		# if coll.is_in_group("Bullet"):
    			# en_hp -= 1
    			
    	if en_hp <= 0 and d_timer == -1:
    		$CollisionShape2D.set_deferred("disabled", true)
    		$AnimatedSprite2D.animation = "Death"
    		$AnimatedSprite2D.play("Death")
    		player.pow_stone += 1
    		constant_vel = 0
    		d_timer = 500
    	if d_timer == 0:
    		queue_free()
    		
    			
    
