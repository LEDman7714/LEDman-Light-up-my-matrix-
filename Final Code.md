###  Видео:
https://drive.google.com/drive/folders/1t-OD45QBLOPZ8_mfcNcZrWXxK8V8EbTb

###  Код: 

### Игрок:

    extends CharacterBody2D

    class_name Player
    
    
    var manager
    @onready var workshop = $CanvasLayer/Workshop
    @export var speed = 500 #скорость игрока
    @export var destination : String = "res://scenes/Levels/levels_menu.tscn"
    
    
    var weapon = Weapon.new() 
    var weapon2 = Weapon.new() 
    var weapon_global = weapon
    var freeze = Weapon.create("Freeze")
    var firegun = Weapon.create("FireGun")
    var sniperrifle = Weapon.create("SniperRifle")
    var cooldown = 0   #Переменная перезарядки
    var screen
    var fire_angle : float = 0      #Переменная угла стрельбы
    var hp : int = 100       #Переменная здоровья игрока
    var eng : int = 250       #Переменная текущей энергии
    var max_eng : int = 250    #Переменная максимальной энергии
    var acceleration : float = 0
    var s_frames : int = 0
    var dmg_effect : int = 0
    var eng_reload : int = 0
    var global
    # var ablt_coldown : int = 0
    var resistance = 0
    
    var i = 1
    
    # condition
    var interact : bool = false
    var interact_cooldown = 0
    var portal_entr : bool = false
    var ard_int : bool = false
    
    # @onready var chip_en = get_node("res://scenes/chip_enemy.tscn")
    # items
    # var pow_stone : int = 0
    # var details : int = 0
    
    var curr_ps = 0
    var curr_d = 0
    var curr_cw = 0
    var curr_ard = 0
    var curr_w = 0
    var curr_l = 0
    var curr_ld = 0
    
    # Called when the node enters the scene tree for the first time.
    func _ready():
    	global = get_node("/root/Global")
    	screen = get_viewport_rect().size
    	Engine.max_fps = 120          #Максимальное количество кадров в секунду проекта
    	$AnimatedSprite2D.play("Idle1")    #Объявление анимации по умолчанию
    	$WeaponSprite.play("default")
    	
    	curr_ps = 0
    	curr_d = 0
    	curr_cw = 0
    	curr_ard = 0
    	curr_w = 0
    	curr_l = 0
    	curr_ld = 0
    
    func _process(_delta):
    	workshop.hide()
    	if interact_cooldown > 0:
    		interact_cooldown -= 1
    	else:
    		interact_cooldown = 0
    	
    	if velocity.length() > 0:
    		$AnimatedSprite2D.animation = "Walk1"     #Если скорость больше 0 про0,игрывается анимация ходьбы
    	elif velocity.length() <= 0 and hp > 0:
    		$AnimatedSprite2D.animation = "Idle1"
    	elif hp <= 0:
    		speed = 0
    		$AnimatedSprite2D.animation = "Death"
    		get_node("CollisionShape2D").set_deferred("disabled", true)
    		get_node("WeaponSprite").hide()
    		cooldown = -1         
    		eng_reload = -1
    	var key_main_menu = Input.is_action_just_pressed("mainmenu")
    	if key_main_menu:
    		get_tree().change_scene_to_file("res://scenes/Main_Menu.tscn")
    	fire_angle = get_local_mouse_position().angle()     #Получение значения угла между осью Х и прямой проходящей через центр игрока и курсор мыши для настройки стрельбы
    	$WeaponSprite.sprite_frames = weapon.texture
    	$WeaponSprite.play("default")
    	$WeaponSprite.rotation = fire_angle
    	if dmg_effect > 0:
    		hp -= dmg_effect
    		dmg_effect = 0
    	queue_redraw()
    	
    	if Input.is_action_just_pressed("Cheat"):
    		global.global_pow_stone += 50
    		global.global_details += 115
    		global.global_copper_wires += 103
    
    
    func _physics_process(_delta):
    	if resistance > 0:
    		resistance -= 1
    		dmg_effect = 0
    	
    	if cooldown > 0:     #Перезарядка орудия привязана к игроку
    		cooldown -= 1
    	if eng_reload > 0:
    		eng_reload -= 1
    	if eng_reload == 0 and eng < 250:
    		eng += 1
    		eng_reload = 30
    	# if ablt_coldown > 0:
    	# 	ablt_coldown -= 1
    	# if ablt_coldown == 0:
    	# 	var ablt = Input.is_action_pressed("ability")
    		
    	
    	if hp <= 0:
    		if i == 1:
    			global.stat_deaths += 1
    			i = 0
    		speed = 0
    		$AnimatedSprite2D.animation = "Death"
    		get_node("CollisionShape2D").set_deferred("disabled", true)   
    		
    		
    	velocity = Vector2.ZERO
    	var key_up = Input.is_action_pressed("up")    #Назначение клавиш поведения игрока:
    	var key_down = Input.is_action_pressed("down")
    	var key_left = Input.is_action_pressed("left")
    	var key_right = Input.is_action_pressed("right")
    	var h_sign = int(key_right) - int(key_left)
    	var v_sign = int(key_down) - int(key_up)
    	velocity = Vector2(h_sign, v_sign).normalized() * speed     #Назначение скорости игрока с использованием нормализации чтобы игрок ходил во все стороны с одной скоросью
    	if h_sign != 0:
    		$AnimatedSprite2D.flip_h = h_sign < 0      #Отражение Спрайта Игрока по оси Y при смене направления движения
    	$WeaponSprite.flip_v = abs(fire_angle) > PI / 2
    	if Input.is_mouse_button_pressed(MOUSE_BUTTON_LEFT): #Клавиша стрельбы
    		attack()
    		print(Engine.get_frames_per_second())    #Лог Кадров в секунду
    	if weapon_global != weapon:
    		if Input.is_action_just_pressed("weapon_change"):
    			var temp_weapon : Weapon = weapon
    			weapon = weapon2
    			weapon2 = temp_weapon
    	
    
    		
    		
    	move_and_slide()
    	for i in get_slide_collision_count():
    		var coll = get_slide_collision(i).get_collider()
    		# enemies
    		if coll.is_in_group("Enemy"):
    			if resistance == 0:
    				if coll.is_in_group("Resistor"):
    					dmg_effect += 1
    					resistance += 10
    				if coll.is_in_group("Chip"):
    					dmg_effect += 2 
    					resistance += 10
    				if coll.is_in_group("Indicator"):
    					dmg_effect += 4
    					resistance += 10
    		if coll.is_in_group("Pila"):
    				dmg_effect += 1
    		# interaction
    		if coll.is_in_group("Portal"):
    			portal_entr = true
    		if coll.is_in_group("workshop"):
    			if interact_cooldown == 0:
    				interact = true
    				workshop.show()
    				get_tree().paused = true
    		else:
    			workshop.hide()
    		if coll.is_in_group("Arduino"):
    			ard_int = true
    		else:
    			ard_int = false
    		# guns
    		if coll.is_in_group("Freeze_Gun"):
    			weapon = freeze
    		if coll.is_in_group("Fire_Gun"):
    			weapon = firegun
    		if coll.is_in_group("Sniper_Rifle"):
    			weapon = sniperrifle
    
    
    func attack():           #Функция Атаки
    	if cooldown == 0 and eng >= weapon.eng:   #проверка перезарядки и наличия энергии для инициализации выстрела
    		cooldown = weapon.cooldown            #перезарядка орудия
    		eng -= weapon.eng            #Вычитание энергии из общего количества при выстреле
    		if weapon.weapon_name == "Goyda":     #проверка оружия
    			print("GOYDA SHOOT")        #Лог выстрелов
    			var bullet = preload("res://Objects/Weapons/bullet.tscn").instantiate()        #Объявление пули в сцене
    			var inacc : float = randf_range(-weapon.inaccuracy / 2, weapon.inaccuracy / 2)     #настройка дуги разброса снарядов конкретно для этого орудия
    			add_sibling(bullet)
    			bullet.position = position
    			bullet.linear_velocity = Vector2(1000, 0).rotated(fire_angle + inacc)     #назначение скорости снаряда и его угла полета
    			bullet.constant_vel = 1000
    			bullet.max_b = 4
    		elif weapon.weapon_name == "Freeze":     #проверка оружия
    			print("FREEZE SHOOT")        #Лог выстрелов
    			var bullet = preload("res://Objects/Weapons/snejinka.tscn").instantiate()        #Объявление пули в сцене
    			var inacc : float = randf_range(-weapon.inaccuracy / 2, weapon.inaccuracy / 2)     #настройка дуги разброса снарядов конкретно для этого орудия
    			add_sibling(bullet)
    			bullet.position = position
    			bullet.linear_velocity = Vector2(400, 0).rotated(fire_angle + inacc)     #назначение скорости снаряда и его угла полета
    			bullet.constant_vel = 400
    			bullet.max_b = 0
    		elif weapon.weapon_name == "FireGun":     #проверка оружия
    			print("Fire")        #Лог выстрелов
    			var bullet = preload("res://scenes/Player/fire.tscn").instantiate()        #Объявление пули в сцене
    			var inacc : float = randf_range(-weapon.inaccuracy / 2, weapon.inaccuracy / 2)     #настройка дуги разброса снарядов конкретно для этого орудия
    			add_sibling(bullet)
    			bullet.position = position
    			bullet.linear_velocity = Vector2(700, 0).rotated(fire_angle + inacc)     #назначение скорости снаряда и его угла полета
    			bullet.constant_vel = 700
    			bullet.max_b = 1#weapon.max_b
    		elif weapon.weapon_name == "SniperRifle":     #проверка оружия
    			print("Sniper")        #Лог выстрелов
    			var bullet = preload("res://scenes/Weapons/sniper_bullet.tscn").instantiate()        #Объявление пули в сцене
    			var inacc : float = randf_range(-weapon.inaccuracy / 2, weapon.inaccuracy / 2)     #настройка дуги разброса снарядов конкретно для этого орудия
    			add_sibling(bullet)
    			bullet.position = position
    			bullet.linear_velocity = Vector2(1400, 0).rotated(fire_angle + inacc)     #назначение скорости снаряда и его угла полета
    			bullet.constant_vel = 1400
    			bullet.max_b = 1
    			
    
    
    func _on_draw() -> void:
    	draw_arc(Vector2.ZERO, 100, -weapon.inaccuracy / 2 + fire_angle, weapon.inaccuracy / 2 + fire_angle, 15, Color.BLACK, 4, false)    #Отрисовка прицела - дуги разброса снарядов


### Враги созданы практически по одному и тому же паттерну, и различия небольшие:

    extends CharacterBody2D
    var damage : int = 4
    var en_hp : int = 8
    var radius : float = 512
    var player : Player
    var global
    var level
    var constant_vel : float = 200
    var acceleration : float = 0
    var s_frames : int = 0
    var d_timer : int = -1
    var r = 255
    var g = 255
    var b = 255
    var random_cw
    # Effects
    var burn = 0
    func _ready() -> void:
    	random_cw = randi_range(1, 3)
    	player = get_tree().get_first_node_in_group("Player")
    	global = get_node("/root/Global")
    	level = get_tree().get_first_node_in_group("level")
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
    		if randi_range(2, 20) == 1:
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
    		global.global_pow_stone += 1
    		global.global_copper_wires += random_cw
    		global.stat_res += (1 + random_cw)
    		global.stat_kills += 1
    		player.curr_ps += 1
    		player.curr_cw += random_cw
    		level.enemies -= 1
    		constant_vel = 0
    		d_timer = 500
    	if d_timer == 0:
    		queue_free()


### Код с данными оружий игрока:

	extends Resource
	
	class_name Weapon
	var weapon_name : String = "Goyda"  #Название дефолтного оружия
	var texture : SpriteFrames = load("res://LEDman/sprites/Guns/sf_sniperrifle.tres")
	var bullet_texture : SpriteFrames = load("res://Objects/Weapons/bullet.tscn::SpriteFrames_ygvmd")   #Загрузка текстуры пули
	var type : String = "Ranged"   
	var eng : int = 1    #Затраты энергии на выстрел
	var eng_chance : float = 1.0
	var dmg : int = 2    #Урон от попадания
	var cooldown : int = 20   #Скорость перезарядки (0.5 секунды)
	var inaccuracy : float = PI / 12    #Угол разброса снарядов для этого орудия
	var max_b : int = 4     #Количество отскоков, которые может совершить снаряд до исчезновения
	
	static func create(nam : String):
		var res = Weapon.new()     #Объявление и настройка дополнительного оружия
		if nam == "Goyda":
			res.texture = load("res://LEDman/Sprites/Guns/Goyda.png")
			res.weapon_name = nam
			res.type = "Ranged"
			res.eng = 1
			res.eng_chance = 1.0
			res.dmg = 2
			res.cooldown = 20
			res.inaccuracy = PI / 12
			res.max_b = 0
		if nam == "Freeze":
			res.texture = load("res://LEDman/Sprites/Guns/sf_freeze.tres")
			res.weapon_name = nam
			res.type = "Ranged"
			res.eng = 1
			res.eng_chance = 0.2
			res.dmg = 3
			res.cooldown = 2
			res.inaccuracy = PI / 4
			res.max_b = 0
		elif nam == "SniperRifle":
			res.texture = load("res://LEDman/Sprites/Guns/Sniper_Rifle.tres")
			res.bullet_texture = load("res://Objects/Weapons/Sniper_bullet_anim.tres")
			res.weapon_name = nam
			res.type = "Ranged"
			res.eng = 4
			res.dmg = 8
			res.cooldown = 80
			res.inaccuracy = PI / 48
			#res.max_b = 0
		elif nam == "FireGun":
			res.texture = load("res://LEDman/Sprites/Guns/FireGun.tres")
			res.bullet_texture = load("res://LEDman/Sprites/Bullets/Fire_sprite_frames.tres")
			res.weapon_name = nam
			res.type = "Ranged"
			res.eng = 1
			res.dmg = 1
			res.cooldown = 2
			res.inaccuracy = PI / 7
			#res.max_b = 0
		return res


### Код оружий как лежачих объектов на уровне, которые можно поднять:

	extends AnimatableBody2D
	var player = Player
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		player = get_tree().get_first_node_in_group("Player")
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		if player.weapon == player.freeze:
			$CollisionShape2D.set_deferred("disabled", true)
			$AnimatedSprite2D.hide()
		elif player.weapon2 != player.freeze:
			$CollisionShape2D.set_deferred("disabled", false)
			$AnimatedSprite2D.show()


### Код пули начального оружия и снайперской винтовки отличается только разным уроном и количеством возможных отскоков:

	extends RigidBody2D
	
	var d_timer : int = 1200     #Таймер до исчезновения снаряда 
	var constant_vel : float = 1000    #Скорость полета снаряда орудия
	var max_b : int = 0  
	
	
	func _ready() -> void:
		pass # Replace with function body.
	
	
	
	func _process(delta: float) -> void:
		if d_timer > 0:        #Функция исчезновения снаряда:
			d_timer -= 1
		elif d_timer == 0:
			queue_free()      #Если таймер исчезновения дошел до 0, снаряд исчезает
		rotation = linear_velocity.angle()   #Поворот снаряда в зависимости от угла его запуска
	
	
	func _on_body_entered(body: Node) -> void:    #Функция столкновения
		if body.is_in_group("Wall"):         #Если объект столкновения - стена
			linear_velocity *= 1 / linear_velocity.length() * constant_vel    #Угол падения равен углу отражения
			if max_b > 0:  #При этом количество оставшихся отскоков уменьшается на 1:
				max_b -= 1
			else:          #Если отскоков не осталось Снаряд пропадет при следующем столкновении со стеной
				queue_free()
		if body.is_in_group("Enemy"):
			body.en_hp -= 2
			var dmg_number = preload("res://scenes/UI/damage_number.tscn").instantiate()
			dmg_number.text = str(2)
			add_sibling(dmg_number)
			dmg_number.global_position = body.global_position
			queue_free()
		if body.is_in_group("Breaking Wall"):
			body.endurance -= 1
			if body.endurance == 0:
				body.get_node("CollisionShape2D").set_deferred("disabled", true)
				body.modulate = Color8(0, 0, 0, 0)
			queue_free()
		if body.is_in_group("Pila"):
			queue_free()

### Код пули замораживающего оружия:

	extends RigidBody2D
	
	var d_timer : int = 140      #Таймер до исчезновения снаряда 
	var constant_vel : float = 100    #Скорость полета снаряда орудия
	var max_b : int = 0  
	var r = 255
	
	
	func _ready() -> void:
		pass # Replace with function body.
	
	
	
	func _process(delta: float) -> void:
		pass
	
	
	func _physics_process(delta: float) -> void:
		if d_timer > 0:        #Функция исчезновения снаряда:
			d_timer -= 1
		elif d_timer == 0:
			queue_free()      #Если таймер исчезновения дошел до 0, снаряд исчезает
		rotation = linear_velocity.angle()   #Поворот снаряда в зависимости от угла его запуска
	
	
	func _on_body_entered(body: Node) -> void:    #Функция столкновения
		if body.is_in_group("Wall"):    
			linear_velocity *= 1 / linear_velocity.length() * constant_vel     #Если объект столкновения - стена
			if max_b >= 0:  #При этом количество оставшихся отскоков уменьшается на 1:
				max_b -= 1
			else:          #Если отскоков не осталось Снаряд пропадет при следующем столкновении со стеной
				queue_free()
		if body.is_in_group("Enemy"): 
			if body.constant_vel > 20:
				body.constant_vel -= 20
			body.r -= 50
			body.modulate = Color8(r, 255, 255)
			body.en_hp -= 1
			var dmg_number = preload("res://scenes/UI/damage_number.tscn").instantiate()
			dmg_number.text = str(1)
			dmg_number.modulate = Color8(0, 231, 215, 255)
			add_sibling(dmg_number)
			dmg_number.global_position = body.global_position
			queue_free()

### Код огненных частиц огнемета:

	extends RigidBody2D
	
	var d_timer : int = 75     #Таймер до исчезновения снаряда 
	var constant_vel : float = 100    #Скорость полета снаряда орудия
	var max_b : int = 0  
	var g = 255
	var b = 255
	
	func _ready() -> void:
		pass # Replace with function body.
	
	
	
	func _process(delta: float) -> void:
		pass
	
	
	func _physics_process(delta: float) -> void:
		if d_timer > 0:        #Функция исчезновения снаряда:
			d_timer -= 1
		elif d_timer == 0:
			queue_free()      #Если таймер исчезновения дошел до 0, снаряд исчезает
		rotation = linear_velocity.angle()   #Поворот снаряда в зависимости от угла его запуска
	
	
	func _on_body_entered(body: Node) -> void:    #Функция столкновения
		if body.is_in_group("Wall"):    
			queue_free()
		if body.is_in_group("Enemy"): 
			if g > 120:
				body.g -= 20
				body.b -= 30
			body.en_hp -= 1
			body.burn += 10
			var dmg_number = preload("res://scenes/UI/damage_number.tscn").instantiate()
			dmg_number.text = str(1)
			dmg_number.modulate = Color8(255, 94, 0, 255)
			add_sibling(dmg_number)
			dmg_number.global_position = body.global_position
			queue_free()


### Код портала: 

	extends Node2D
	var player : Player
	var level
	var label
	@export var destination : String = "res://scenes/Levels/levels_menu.tscn"
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		level = get_tree().get_first_node_in_group("level")
		player = get_tree().get_first_node_in_group("Player")
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		if level.enemies == 0:
			get_node("CollisionShape2D").set_deferred("disabled", false)
		else:
			get_node("CollisionShape2D").set_deferred("disabled", true)


### В коде каждого уровня прописано только количество противников:

	extends Node2D
	var enemies : int = 36
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		pass # Replace with function body.
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		pass


### Код пилы:

	extends CharacterBody2D
	var player : Player
	var damage : int = 2
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		player = get_tree().get_first_node_in_group("Player")
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		pass
		
	func _physics_process(_delta):
		for i in get_slide_collision_count():
			var coll = get_slide_collision(i).get_collider()
			if coll.is_in_group("Player"):
				player.hp -= 2
		
### Код ломающейся стены

	extends AnimatableBody2D
	var start_pos : Vector2
	var r : int = 1
	var g : int = 1
	var b : int = 1
	var scl : float = 1
	var player : Player
	var endurance : int = 2
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		player = get_tree().get_first_node_in_group("Player")
		start_pos = global_position
		var col_ch = Color8(12, 120, 1, 255)
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		var pl = player.global_position
		var distance = (pl - start_pos).length()
		#rotation = (mous - start_pos).angle()
		modulate  = Color8(r, g, b, 200)
		g = 21000/distance - 7
		b = 30000/distance
		if endurance == 0:
			modulate = Color8(0, 0, 0, 0)
		# move_and_collide(Vector2.ZERO)
  

  
### Код с глобальными переменными:

	extends Node2D
	var global_pow_stone : int = 0
	var global_details : int = 0
	var global_copper_wires : int = 0
	
	var global_wires : int = 0
	var global_arduino : int = 0
	var global_laser : int = 0
	var global_layout : int = 0
	
	# statistics
	var stat_kills : int = 0
	var stat_deaths : int = 0
	var stat_boss_kills : int = 0
	var stat_levels_completed : int = 0
	var stat_res : int = 0
	
	
	var player : Player
	var CRT 
	var condition : int = 0
	var crt : int = 0
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		print('Global is ready')
		player = get_tree().get_first_node_in_group("Player")
		CRT = get_node("res://UI/CRT.tscn")
		
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		pass


  ### Код так называемого Менеджера, который управляет кнопками паузы и кнопками внутри объектов, с которыми может взаимодействовать игрок:

	  extends Node
	var game_paused : bool = false
	var player = Player
	var global 
	var end_cooldown : int = 240
	var subd_times = 1
	
	var r = 255
	var g = 255
	var b = 255
	var r1 = 255
	var g1 = 255
	var b1 = 255
	var r2 = 255
	var g2 = 255
	var b2 = 255
	var r3 = 255
	var g3 = 255
	var b3 = 255
	
	@onready var pause_menu = $"../Pause Menu"
	@onready var workshop = $"../Workshop"
	
	@onready var arduino_b = $"../Workshop/Panel/VBoxContainer/Arduino"
	@onready var wires_b = $"../Workshop/Panel/VBoxContainer/Wire"
	@onready var laser_b = $"../Workshop/Panel/VBoxContainer/laser"
	@onready var layout_b = $"../Workshop/Panel/VBoxContainer/Layout Board"
	
	@onready var arduino_l = $"../Workshop/Panel/VBoxContainer/Arduino/Label"
	@onready var wires_l = $"../Workshop/Panel/VBoxContainer/Wire/Label"
	@onready var laser_l = $"../Workshop/Panel/VBoxContainer/laser/Label"
	@onready var layout_l = $"../Workshop/Panel/VBoxContainer/Layout Board/Label"
	
	@onready var end_l = $"../End screen/Panel/End text"
	@onready var end_screen = $"../End screen"
	@onready var pr_l = $"../End screen/Panel/processors2"
	@onready var rs_l = $"../End screen/Panel/resistors"
	@onready var cw_l = $"../End screen/Panel/copper wires"
	
	@export var destination1 : String = "res://scenes/Levels/levels_menu.tscn"
	@export var destination2 : String = "res://scenes/Menues/Settings_Menu.tscn"
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		player = get_tree().get_first_node_in_group("Player")
		global = get_node("/root/Global")
		end_screen.hide()
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		if player.hp <= 0:
			end_cooldown -= 1
		if player.portal_entr == true:
			end_cooldown = 0
			end_l.modulate = Color8(4, 204, 26)
			end_l.set_text("Level Completed!")
			player.speed = 0
			pr_l.text = "Processors: " + str(player.curr_ps)
			rs_l.text = "Resistors Fragments: " + str(player.curr_d)
			cw_l.text = "Copper wires: " + str(player.curr_cw)
		elif player.portal_entr == false and player.hp <= 0:
			end_l.modulate = Color8(255, 21, 98)
			end_l.set_text("Game over")
			if subd_times == 1:
				subd_times = 0
				global.global_pow_stone -= player.curr_ps
				global.global_details -= player.curr_d
				global.global_copper_wires -= player.curr_cw
				global.global_wires -= player.curr_ard
				global.global_arduino -= player.curr_w
				global.global_laser -= player.curr_l
				global.global_layout -= player.curr_ld
			pr_l.text = "Processors: 0"
			rs_l.text = "Resistors Fragments: 0"
			cw_l.text = "Copper wires: 0"
		if end_cooldown == 0:
			end_screen.show()
			
			
		
		if player.interact == false:
			if Input.is_action_just_pressed("ui_cancel"):
				game_paused = !game_paused
			if game_paused == true:
				get_tree().paused = true
				pause_menu.show()
			else:
				get_tree().paused = false
				pause_menu.hide()
		
		if global.global_arduino == 1:
			arduino_b.hide()
		if global.global_layout == 1:
			layout_b.hide()
		if global.global_wires == 3:
			wires_b.hide()
		if global.global_laser == 1:
			laser_b.hide()
			
	func _physics_process(delta: float) -> void:
		g += 3
		g1 += 3
		g2 += 3
		g3 += 3
		b += 3
		b1 += 3
		b2 += 3
		b3 += 3
		arduino_l.modulate = Color8(r, g, b)
		wires_l.modulate = Color8(r1, g1, b1)
		laser_l.modulate = Color8(r2, g2, b2)
		layout_l.modulate = Color8(r3, g3, b3)
			
	
	
	func _on_resume_pressed() -> void:
		print("aboba")
		game_paused = !game_paused
	
	
	func _on_menu_pressed() -> void:
		if player.portal_entr == true:
			global.stat_levels_completed += 1
		if player.portal_entr == false:
			global.global_pow_stone -= player.curr_ps
			global.global_details -= player.curr_d
			global.global_copper_wires -= player.curr_cw
			global.global_wires -= player.curr_ard
			global.global_arduino -= player.curr_w
			global.global_laser -= player.curr_l
			global.global_layout -= player.curr_ld
		get_tree().paused = false
		get_tree().change_scene_to_file(destination1)
		player.curr_ps = 0
		player.curr_d = 0
		player.curr_cw = 0
		player.curr_ard = 0
		player.curr_w = 0
		player.curr_l = 0
		player.curr_ld = 0
	
	
	func _on_settings_pressed() -> void:
		if player.portal_entr == false:
			global.global_pow_stone -= player.curr_ps
			global.global_details -= player.curr_d
			global.global_copper_wires -= player.curr_cw
			global.global_wires -= player.curr_ard
			global.global_arduino -= player.curr_w
			global.global_laser -= player.curr_l
			global.global_layout -= player.curr_ld
		get_tree().paused = false
		get_tree().change_scene_to_file(destination2)
		player.curr_ps = 0
		player.curr_d = 0
		player.curr_cw = 0
		player.curr_ard = 0
		player.curr_w = 0
		player.curr_l = 0
		player.curr_ld = 0
	
	
	func _on_arduino_pressed() -> void:
		if global.global_pow_stone >= 40 and global.global_details >= 40 and global.global_copper_wires >= 40:
			global.global_arduino += 1
			player.curr_ard += 1
			global.global_pow_stone -= 40
			global.global_details -= 40
			global.global_copper_wires -= 40
		else:
			g = 0
			b = 0
			
	func _on_layout_board_pressed() -> void:
		if global.global_details >= 25 and global.global_copper_wires >= 30:
			global.global_layout += 1
			player.curr_ld += 1
			global.global_details -= 25
			global.global_copper_wires -= 30
		else:
			g3 = 0
			b3 = 0
			
			
	func _on_wire_pressed() -> void:
		if global.global_details >= 10 and global.global_copper_wires >= 10:
			global.global_wires += 1
			player.curr_w += 1
			global.global_details -= 10
			global.global_copper_wires -= 10
		else:
			g1 = 0
			b1 = 0
	
	
	func _on_laser_pressed() -> void:
		if global.global_pow_stone >= 10 and global.global_details >= 20 and global.global_copper_wires >= 3:
			global.global_laser += 1
			player.curr_l += 1
			global.global_pow_stone -= 10
			global.global_details -= 20
			global.global_copper_wires -= 3
		else:
			g2 = 0
			b2 = 0
	
	
	func _on_exit_worksop_pressed() -> void:
		player.interact_cooldown = 180
		player.interact = false
		r = 255
		r1 = 255
		r2 = 255
		g = 255
		g1 = 255
		g2 = 255
		b = 255
		b1 = 255
		b2 = 255

  ### Код всех кнопок одинаковый, меняется только место назначения:

  	extends Button

	@export var destination : String = "res://scenes/Levels/boss_level.tscn"
	
	
	# Called when the node enters the scene tree for the first time.
	func _ready():
		pass # Replace with function body.
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta):
		pass
	
	
	func _on_pressed():
		get_tree().change_scene_to_file(destination)


  ### Код объектов фона главного меню и меню настроек:

	  extends TextureRect
	
	var start_pos : Vector2
	var r : int = 1
	var g : int = 1
	var b : int = 1
	var scl : float = 1
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		start_pos = global_position + size / 2 * scale.x
		var col_ch = Color8(12, 120, 1, 255)
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta):
		var mous = get_global_mouse_position()
		var distance = (mous - start_pos).length()
		#rotation = (mous - start_pos).angle()
		modulate  = Color8(r, g, b, 130)
		g = 14000/distance - 14
		b = 20000/distance - 8

  ### Код платы Arduino:

	  extends AnimatableBody2D
	var player = Player
	var global
	var cooldown = 600
	var a = 0
	var ardu = 0
	
	@onready var laser_anim = $Node2D2/Laser/AnimationPlayer
	@onready var ard = $AnimatedSprite2D
	@onready var layout = $"Node2D2/Layout board/LayoutBoard"
	@onready var l_col = $"Node2D2/Layout board/CollisionShape2D"
	@onready var wire1 = $Node2D2/Sprite2D
	@onready var wire2 = $Node2D2/Sprite2D2
	@onready var las_e = $Node2D2/LaserEmitterObj
	@onready var las_l = $Node2D2/Laser
	@onready var las_coll = $Node2D2/Laser/CollisionShape2D
	@onready var labels = $Control
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		player = get_tree().get_first_node_in_group("Player")
		global = get_node("/root/Global")
		labels.hide()
		# hide()
		layout.hide()
		wire1.hide()
		wire2.hide()
		las_l.hide()
		ard.hide()
		las_e.hide()
		las_coll.set_deferred("disabled", true)
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
			if player.ard_int == true:
				if global.global_layout == 1:
					layout.show()
					ardu += 1
					a += 1
				if global.global_laser == 1 and a == 1:
					las_e.show()
					ardu += 1
				if global.global_arduino == 1:
					labels.show()
					ard.show()
					ardu += 1
				if global.global_wires == 3 and a == 1:
					wire1.show()
					wire2.show()
					ardu += 1
					
					
	func _physics_process(delta: float) -> void:
		if ardu >= 4:
			cooldown -= 1
			if cooldown <= 0:
				las_l.show()
				laser_anim.play("Laser")
				las_coll.set_deferred("disabled", false)
			if cooldown == -600:
				cooldown = 600
			if cooldown > 0:
				las_coll.set_deferred("disabled", true)

### Код Лазера, выпускаемого лазерным модулем:

	extends CharacterBody2D
	var boss
	var damage : int = 8
	
	
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		boss = get_tree().get_first_node_in_group("Boss")
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta: float) -> void:
		pass
		
	func _physics_process(_delta):
		for i in get_slide_collision_count():
			var coll = get_slide_collision(i).get_collider()
			if coll.is_in_group("Boss"):
				boss.en_hp -= 8


### Код Всех надписей тоже практически одинаковый:

	extends Label
	
	var player : Player
	# Called when the node enters the scene tree for the first time.
	func _ready() -> void:
		player = get_tree().get_first_node_in_group("Player")
		print(player.hp)
	
	
	# Called every frame. 'delta' is the elapsed time since the previous frame.
	func _process(delta):
		if player.hp > 0:
			text = str(player.hp) + '/100'
		if player.hp <= 0:
			text = '0/100'


### Код (шейдер) эффекта лучевой трубки:

	shader_type canvas_item;
	
	uniform bool overlay = false;
	
	uniform sampler2D SCREEN_TEXTURE : hint_screen_texture, filter_linear_mipmap;
	
	uniform float scanlines_opacity : hint_range(0.0, 1.0) = 0.4;
	uniform float scanlines_width : hint_range(0.0, 0.5) = 0.25;
	uniform float grille_opacity : hint_range(0.0, 1.0) = 0.3;
	uniform vec2 resolution = vec2(640.0, 480.0); // Rоличество строк и столбцов, на которые будет разделена текстура (получатся маленькие квадраты)
	
	uniform bool pixelate = true; // Каждый квадрат заполняется цветом создавая эффект пикселизации
	
	uniform bool roll = true;
	uniform float roll_speed = 8.0;
	uniform float roll_size : hint_range(0.0, 100.0) = 15.0;
	uniform float roll_variation : hint_range(0.1, 5.0) = 1.8;
	uniform float distort_intensity : hint_range(0.0, 0.2) = 0.05; // эффект искажения
	
	uniform float noise_opacity : hint_range(0.0, 1.0) = 0.4;
	uniform float noise_speed = 5.0; // скорость изменения шума
	
	uniform float static_noise_intensity : hint_range(0.0, 1.0) = 0.06;
	
	uniform float aberration : hint_range(-1.0, 1.0) = 0.03; // эффект аберрации
	uniform float brightness = 1.4; // яркость экрана
	uniform bool discolor = true; // эффект чб (обесцвечивание)
	
	uniform float warp_amount :hint_range(0.0, 5.0) = 1.0; // эффект вытягивания как у старых телевизоров
	uniform bool clip_warp = false;
	
	uniform float vignette_intensity = 0.4; // виньетка (затемнение по краям экрана)
	uniform float vignette_opacity : hint_range(0.0, 1.0) = 0.5;
	
	// шум создается с помощью генерации псевдослучайных чисел между 0.0 и 1.0
	vec2 random(vec2 uv){
	    uv = vec2( dot(uv, vec2(127.1,311.7) ),
	               dot(uv, vec2(269.5,183.3) ) );
	    return -1.0 + 2.0 * fract(sin(uv) * 43758.5453123);
	}
	

	float noise(vec2 uv) {
	    vec2 uv_index = floor(uv);
	    vec2 uv_fract = fract(uv);
	
	    vec2 blur = smoothstep(0.0, 1.0, uv_fract);
	
	    return mix( mix( dot( random(uv_index + vec2(0.0,0.0) ), uv_fract - vec2(0.0,0.0) ),
	                     dot( random(uv_index + vec2(1.0,0.0) ), uv_fract - vec2(1.0,0.0) ), blur.x),
	                mix( dot( random(uv_index + vec2(0.0,1.0) ), uv_fract - vec2(0.0,1.0) ),
	                     dot( random(uv_index + vec2(1.0,1.0) ), uv_fract - vec2(1.0,1.0) ), blur.x), blur.y) * 0.5 + 0.5;
	}
	
	// Поглощает ультрафиолетовое излучение и деформирует края, создавая эффект сферизации
	vec2 warp(vec2 uv){
		vec2 delta = uv - 0.5;
		float delta2 = dot(delta.xy, delta.xy);
		float delta4 = delta2 * delta2;
		float delta_offset = delta4 * warp_amount;
	
		return uv + delta * delta_offset;
	}
	
	// Добавляет черную рамку, чтобы скрыть растянутый пиксель, созданный эффектом деформации
	float border (vec2 uv){
		float radius = min(warp_amount, 0.08);
		radius = max(min(min(abs(radius * 2.0), abs(1.0)), abs(1.0)), 1e-5);
		vec2 abs_uv = abs(uv * 2.0 - 1.0) - vec2(1.0, 1.0) + radius;
		float dist = length(max(vec2(0.0), abs_uv)) / radius;
		float square = smoothstep(0.96, 1.0, dist);
		return clamp(1.0 - square, 0.0, 1.0);
	}
	
	// Добавляет тень-виньетку к краям изображения
	float vignette(vec2 uv){
		uv *= 1.0 - uv.xy;
		float vignette = uv.x * uv.y * 15.0;
		return pow(vignette, vignette_intensity * vignette_opacity);
	}
	
	void fragment()
	{
		vec2 uv = overlay ? warp(SCREEN_UV) : warp(UV); // Warp the uv. uv will be used in most cases instead of UV to keep the warping
		vec2 text_uv = uv;
		vec2 roll_uv = vec2(0.0);
		float time = roll ? TIME : 0.0;
	
	
		// Создание эффекта пикселизации текстуры в зависимости от заданного разрешения
		if (pixelate)
		{
			text_uv = ceil(uv * resolution) / resolution;
		}
	
		
		float roll_line = 0.0;
		if (roll || noise_opacity > 0.0)
		{
			roll_line = smoothstep(0.3, 0.9, sin(uv.y * roll_size - (time * roll_speed) ) );
			roll_line *= roll_line * smoothstep(0.3, 0.9, sin(uv.y * roll_size * roll_variation - (time * roll_speed * roll_variation) ) );
			roll_uv = vec2(( roll_line * distort_intensity * (1.-UV.x)), 0.0);
		}
	
		vec4 text;
		if (roll)
		{
			// Мы применяем аберрацию к красному и зеленому каналам в соответствии с параметром aberration
			text.r = texture(SCREEN_TEXTURE, text_uv + roll_uv * 0.8 + vec2(aberration, 0.0) * .1).r;
			text.g = texture(SCREEN_TEXTURE, text_uv + roll_uv * 1.2 - vec2(aberration, 0.0) * .1 ).g;
			text.b = texture(SCREEN_TEXTURE, text_uv + roll_uv).b;
			text.a = 1.0;
		}
		else
		{
			// Иначе аберрация не применяется, поэтому коэффициент равен .1
			text.r = texture(SCREEN_TEXTURE, text_uv + vec2(aberration, 0.0) * .1).r;
			text.g = texture(SCREEN_TEXTURE, text_uv - vec2(aberration, 0.0) * .1).g;
			text.b = texture(SCREEN_TEXTURE, text_uv).b;
			text.a = 1.0;
		}
	
		float r = text.r;
		float g = text.g;
		float b = text.b;
	
		uv = warp(UV);
	
		// ЭЛТ-мониторы содержат не пиксели, а группы красных, зеленых и синих точек или линий, называемых решеткой. Мы выделяем цветовые каналы текстуры и разделяем их тремя смещенными линиями, чтобы показать красный, зеленый и синий цвета рядом друг с другом, с небольшим черным промежутком между ними.
		if (grille_opacity > 0.0){
	
			float g_r = smoothstep(0.85, 0.95, abs(sin(uv.x * (resolution.x * 3.14159265))));
			r = mix(r, r * g_r, grille_opacity);
	
			float g_g = smoothstep(0.85, 0.95, abs(sin(1.05 + uv.x * (resolution.x * 3.14159265))));
			g = mix(g, g * g_g, grille_opacity);
	
			float b_b = smoothstep(0.85, 0.95, abs(sin(2.1 + uv.x * (resolution.x * 3.14159265))));
			b = mix(b, b * b_b, grille_opacity);
	
		}
	
		text.r = clamp(r * brightness, 0.0, 1.0);
		text.g = clamp(g * brightness, 0.0, 1.0);
		text.b = clamp(b * brightness, 0.0, 1.0);
	
		// Скан-линии - это горизонтальные линии, из которых состоит изображение на электронно-лучевом мониторе.
		// Здесь мы фактически устанавливаем черный промежуток между каждой строкой
		float scanlines = 0.5;
		if (scanlines_opacity > 0.0)
		{
			// Используя ту же технику, что и выше, тут мы создаем синусоидные линии
			scanlines = smoothstep(scanlines_width, scanlines_width + 0.5, abs(sin(uv.y * (resolution.y * 3.14159265))));
			text.rgb = mix(text.rgb, text.rgb * vec3(scanlines), scanlines_opacity);
		}
	
		// Применение полосчатого шума.
		if (noise_opacity > 0.0)
		{
			// Создание сильно растянутого эффекта шума для создания "шумных" полос
			float noise = smoothstep(0.4, 0.5, noise(uv * vec2(2.0, 200.0) + vec2(10.0, (TIME * (noise_speed))) ) );

			roll_line *= noise * scanlines * clamp(random((ceil(uv * resolution) / resolution) + vec2(TIME * 0.8, 0.0)).x + 0.8, 0.0, 1.0);
			text.rgb = clamp(mix(text.rgb, text.rgb + roll_line, noise_opacity), vec3(0.0), vec3(1.0));
		}
	
		// Добавление статичного шума
		if (static_noise_intensity > 0.0)
		{
			text.rgb += clamp(random((ceil(uv * resolution) / resolution) + fract(TIME)).x, 0.0, 1.0) * static_noise_intensity;
		}
	
		// Добавление черной рамки создающей эффект телефизора
		text.rgb *= border(uv);
		text.rgb *= vignette(uv);
		if (clip_warp)
		{
			text.a = border(uv);
		}
	
		// Изменение насыщенности и контрастности
		float saturation = 0.5;
		float contrast = 1.2;
		if (discolor)
		{
			// Насыщенность
			vec3 greyscale = vec3(text.r + text.g + text.b) / 3.;
			text.rgb = mix(text.rgb, greyscale, saturation);
	
			// Контраст
			float midpoint = pow(0.5, 2.2);
			text.rgb = (text.rgb - vec3(midpoint)) * contrast + midpoint;
		}
	
		COLOR = text;
	}


     

    	
