# ui
## vim.ui.select

```lua
local themes = { "one-dark", "tokyo-night", "vscode" }
local options = {
	prompt = "Выбери из списка",
	format_item = function(item) 
		return "Темы: " .. item
	end
}
local action = function(theme)
	vim.cmd("colorscheme " .. theme)	
end

vim.ui.select(themes, options, action)
```

Параметры:
- _items_ - Массив значений
- _options_ - Дополнительные опции:
	- _prompt_ - Заголовок списка
	- _format_item_ - Функция которая форматирует подсказки для ui интерфейса
- _action_ - Функция, которая вызывается один раз, когда пользователь сделал выбор.

# fn
