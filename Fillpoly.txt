extends Node2D

var VetPoint = []
var VetPolly = []
var CorAresta = Color.YELLOW
var CorFillpoly = Color.RED
var AreaDesenho : bool
var CanDraw : bool
var poligonos_preenchidos = {}  # Armazena as linhas de preenchimento para cada polígono
var proximo_indice = 0  # Controla o maior índice já utilizado
var indices_disponiveis = []
var i = 0

@onready var option_button = $Interface/PoligonoSalvo

func _ready():
	CanDraw = false
	pass

func _process(delta: float) -> void:
	if (Input.is_action_just_released("RClick") or Input.is_action_just_pressed("LClick")) and AreaDesenho and CanDraw:
		Pontos()
	pass

func Pontos():
	var MousePos = get_global_mouse_position()
	VetPoint.append(MousePos)
	queue_redraw()

func Polly():
	if VetPoint.size() > 2:
		# Armazena os pontos do polígono e a cor das arestas
		VetPolly.append({"pontos": VetPoint.duplicate(), "cor": CorAresta})
		VetPoint = []
		queue_redraw()
		CanDraw = false
	pass

func _draw() -> void:
	# Desenhar as arestas e círculos do polígono em criação
	for i in range(len(VetPoint) - 1):
		draw_incremental_line(VetPoint[i], VetPoint[i + 1], CorAresta)
	if len(VetPoint) > 1:
		draw_incremental_line(VetPoint[-1], VetPoint[0], CorAresta)
	for point in VetPoint:
		draw_circle(point, 10, Color.BLACK)
	
	# Desenhar polígonos armazenados
	for poly in VetPolly:
		var pontos = poly["pontos"]
		var cor = poly["cor"]
		for i in range(len(pontos) - 1):
			draw_incremental_line(pontos[i], pontos[i + 1], cor)
		if len(pontos) > 1:
			draw_incremental_line(pontos[-1], pontos[0], cor)
		for point in pontos:
			draw_circle(point, 10, Color.BLACK)

	# Desenhar os preenchimentos de todos os polígonos preenchidos
	for poly_index in poligonos_preenchidos.keys():
		for linha in poligonos_preenchidos[poly_index]:
			draw_line(linha[0], linha[1], linha[2])

func draw_incremental_line(p0, p1, color):
	var x0 = p0.x
	var y0 = p0.y
	var x1 = p1.x
	var y1 = p1.y
	
	var dx = abs(x1 - x0)
	var dy = abs(y1 - y0)

	var sx = 1 if x0 < x1 else -1
	var sy = 1 if y0 < y1 else -1

	var err = dx - dy

	while true:
		draw_circle(Vector2(x0, y0), 5, color)
		if (x0 == x1) and (y0 == y1):
			break
		
		var e2 = err * 2
		if e2 > -dy:
			err -= dy
			x0 += sx
		if e2 < dx:
			err += dx
			y0 += sy

func _on_area_2d_mouse_entered() -> void:
	AreaDesenho = true
	pass

func _on_area_2d_mouse_exited() -> void:
	AreaDesenho = false
	pass

func Preencher(polygon, cor, index):
	if polygon.size() < 3:
		print("Polígono com menos de 3 pontos. Não pode preencher.")
		return
	print("Preenchendo polígono com ", polygon.size(), " pontos.")
	
	var linhas = []  # Linhas de preenchimento para este polígono

	# Encontrar o valor mínimo e máximo de Y
	var min_y = polygon[0].y
	var max_y = polygon[0].y
	for point in polygon:
		if point.y < min_y:
			min_y = point.y
		if point.y > max_y:
			max_y = point.y
	
	# Varra todas as linhas horizontais entre min_y e max_y
	for y in range(int(min_y), int(max_y) + 1):
		var intersecoes = []
		for i in range(len(polygon)):
			var p1 = polygon[i]
			var p2 = polygon[(i + 1) % len(polygon)]  # O último ponto se conecta ao primeiro
			if (p1.y <= y and p2.y > y) or (p2.y <= y and p1.y > y):  # Verifique se cruza o scanline
				var x_inter = p1.x + (y - p1.y) * (p2.x - p1.x) / (p2.y - p1.y)
				intersecoes.append(x_inter)

		intersecoes.sort()  # Organize as interseções da esquerda para a direita
		# Armazene as linhas entre pares de interseções
		for i in range(0, intersecoes.size() - 1, 2):
			linhas.append([Vector2(intersecoes[i], y), Vector2(intersecoes[i + 1], y), cor])

	# Armazena as linhas de preenchimento para este polígono
	poligonos_preenchidos[index] = linhas

	queue_redraw()  # Solicita o redesenho

func _on_novo_poligono_pressed() -> void:
	CanDraw = true
	pass

func _on_cor_arestas_color_changed(color: Color) -> void:
	CorAresta = color
	queue_redraw()

func _on_cor_fillpoly_color_changed(color: Color) -> void:
	CorFillpoly = color
	queue_redraw()

func _on_salvar_pressed() -> void:
	if VetPoint.size() > 2:  
		Polly()  # Chama Polly para salvar os pontos e a cor da aresta
		
		option_button.add_item("Polígono " + str(i))  
		i += 1
	pass

func _on_fill_poly_pressed() -> void:
	var selected_index = option_button.get_selected()
	if selected_index >= 0 and selected_index < VetPolly.size():
		Preencher(VetPolly[selected_index]["pontos"], CorFillpoly, selected_index)
		queue_redraw()
	pass

func _on_limpar_tela_pressed() -> void:
	get_tree().reload_current_scene()
	pass # Replace with function body.

func _on_excluir_poligono_pressed() -> void:
	var selected_index = option_button.get_selected()
	if selected_index >= 0 and selected_index < VetPolly.size():
		option_button.remove_item(selected_index)
		VetPolly.remove_at(selected_index)
		if poligonos_preenchidos.has(selected_index):
			poligonos_preenchidos.erase(selected_index)
		queue_redraw() 
			
	pass # Replace with function body.
