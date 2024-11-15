###  Отчет о работе №1 
Реализован сценарий №3. Создано начальное меню с рабочими кнопками. Можно уже сейчас начать игру, и пройти например первый уровень. Почти полностью дописан код игрока, хотя уже сейчас ходить стрелять и все остальное он может. Так выглядит код игрока:

    extends CharacterBody2D
    
    class_name Player
    
    @export var speed = 500 #скорость игрока
    var weapon = Weapon.new() 
    var weapon22 = Weapon.create("Freeze")
    var weapon2 = Weapon.create("FireGun")
    var cooldown = 0   #Переменная перезарядки
    var screen
    var fire_angle : float = 0      #Переменная угла стрельбы
    var hp : int = 100       #Переменная здоровья игрока
    var eng : int = 250       #Переменная текущей энергии
    var max_eng : int = 250    #Переменная максимальной энергии
    var acceleration : float = 0
    var s_frames : int = 0
    var dmg_effect : int = 0
    
    # enemies
    @onready var chip_en = get_node("res://scenes/chip_enemy.tscn")
    @onready var resistor_en = get_node("res://scenes/Enemy/Resistor.tscn")
    @onready var indicator_en = get_node("res://scenes/Enemy/digital_indicator.tscn")
    # @onready var chip_en = get_node("res://scenes/chip_enemy.tscn")
    # items
    var pow_stone : int = 0
    
    # Called when the node enters the scene tree for the first time.
    func _ready():
    	screen = get_viewport_rect().size
    	Engine.max_fps = 120          #Максимальное количество кадров в секунду проекта
    	$AnimatedSprite2D.play("Idle1")    #Объявление анимации по умолчанию
    	$WeaponSprite.play("default")
    
    func _process(_delta):
    	if velocity.length() > 0:
    		$AnimatedSprite2D.animation = "Walk1"     #Если скорость больше 0 проигрывается анимация ходьбы
    	else:
    		$AnimatedSprite2D.animation = "Idle1"      #Иначе - анимация по умолчанию
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
    
    
    func _physics_process(_delta):
    	if cooldown > 0:     #Перезарядка орудия привязана к игроку
    		cooldown -= 1
    	
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
    	if Input.is_action_just_pressed("weapon_change"):
    		var temp_weapon : Weapon = weapon
    		weapon = weapon2
    		weapon2 = temp_weapon
    	#if Input.is_action_just_pressed("Pause"):
    	
    
    		
    		
    	move_and_slide()
    	for i in get_slide_collision_count():
    		var coll = get_slide_collision(i).get_collider()
    		if coll.is_in_group("Enemy"):
    			dmg_effect += chip_en.damage
    			velocity = velocity * 18
    			acceleration = -0.2
    
    
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
    		elif weapon.weapon_name == "Laser":     #проверка оружия
    			print("LASER SHOOT")        #Лог выстрелов
    			var bullet = preload("res://Objects/Weapons/snejinka.tscn").instantiate()        #Объявление пули в сцене
    			var inacc : float = randf_range(-weapon.inaccuracy / 2, weapon.inaccuracy / 2)     #настройка дуги разброса снарядов конкретно для этого орудия
    			add_sibling(bullet)
    			bullet.position = position
    			bullet.linear_velocity = Vector2(50000, 0).rotated(fire_angle + inacc)     #назначение скорости снаряда и его угла полета
    			bullet.constant_vel = 5000
    			bullet.max_b = 1
    		elif weapon.weapon_name == "FireGun":     #проверка оружия
    			print("Fire")        #Лог выстрелов
    			var bullet = preload("res://scenes/Player/fire.tscn").instantiate()        #Объявление пули в сцене
    			var inacc : float = randf_range(-weapon.inaccuracy / 2, weapon.inaccuracy / 2)     #настройка дуги разброса снарядов конкретно для этого орудия
    			add_sibling(bullet)
    			bullet.position = position
    			bullet.linear_velocity = Vector2(700, 0).rotated(fire_angle + inacc)     #назначение скорости снаряда и его угла полета
    			bullet.constant_vel = 700
    			bullet.max_b = 1#weapon.max_b
    			
    
    
    func _on_draw() -> void:
    	draw_arc(Vector2.ZERO, 100, -weapon.inaccuracy / 2 + fire_angle, weapon.inaccuracy / 2 + fire_angle, 15, Color.BLACK, 4, false)    #Отрисовка прицела - дуги разброса снарядов
    	