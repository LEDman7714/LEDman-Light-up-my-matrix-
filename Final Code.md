###  Видео:
https://drive.google.com/drive/folders/1t-OD45QBLOPZ8_mfcNcZrWXxK8V8EbTb

Код: 
Игрок:

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


Враги созданы порактически по одному и тому же паттерну, и различия небольшие:

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
		
			

    	
