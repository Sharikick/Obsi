`slog` это пакет для логирования в golang

`slog` предоставляет 3 основных типа:

- `Logger` - это интерфейс логов, которые предоставляют методы `Info(), Warn(), Error()` и тд
- `Record` - это представление объекта журнала, созданного `Logger`
- `Handler` -  это интерфейс, который, будучи реализованным определяет форматирование каждого `Record`. Пример `JSONHandler` или `TextHandler`

Для создания логера, существует метод `slog.New`, который принимает реализацию `Handler`

Есть две реализации:

- `NewJSONHandler`
- `NewTextHandler`

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))

logger.Info("Text")
logger.Error("Error")
logger.Warn("Warn")
```

## Добавление атрибутов в запись журнала

`slog.Int`
`slog.String`

```go
logger.Info(
	"Message",
	slog.String("method", "POST"),
	slog.Int(status, 200),	
)
```

## Группировка контекстных атрибутов

Группа создается с помощью `slog.Group`

```go
logger.Info(
	"Message",
	slog.String("method", "POST"),
	slog.Int(status, 200),
	slog.Group(
		"properties",
		slog.Int("width", 200),
		slog.Int("height", 200),	
	)
)
```

## Создание и использование дочерних логеров

Иногда может быть полезно включать одни и те же атрибуты во все записи в определённой области.

У интерфейса `Logger` есть метод `With()`, который принимает логер типа `*slog.Logger` и возвращает логер с добавленными атрибутами

```go
handler := slog.NewJSONHandler(os.Stdout, nil)
logger := slog.New(handler)

child := logger.With(
	slog.Group(
		"program_info",
		slog.Int("pid", os.Getpid()),
		slog.String("go_version", buildInfo.GoVersion),
	),
)
```

## Настройка уровней в `slog`

Пакет `log/slog` предоставляет четыре уровня логирования по умолчанию,

- `DEBUG` (-4) -> `slog.LevelDebug`
- `INFO` (0) -> `slog.LevelInfo`
- `WARN` (4) -> `slog.LevelWarn`
- `ERROR` (8) -> `slog.LevelError`

```go
opts := &slog.HandlerOptions{
	Level: slog.LevelDebug,
}

handler := slog.NewJSONHandler(os.Stdout, nil)
logger := slog.New(handler)

...
```

Если мы хотим динамически менять уровень логирования, то можно сделать через `slog.LevelVar`

```go
logLevel := &slog.LevelVar{} // INFO

opts := &slog.HandlerOptions{
	Level: logLevel,
}

handler := slog.NewJSONHandler(os.Stdout, nil)
logger := slog.New(handler)

...
```

## Создание пользовательских уровней журнала

Если вам нужны собственные уровни, помимо тех, что Slog предоставляет по умолчанию, вы можете создать их через интерфейс `Leveler`

```go
type Leveler interface {
	Level() Level
}
```

Этот интерфейс легко реализовать через тип `Level`

```go
const (
	LevelTrace = slog.Level(-8)
	LevelFatal = slog.Level(12)
)
```

Определив пользовательские уровни, как указано выше, вы можете использовать их только через метод `Log()` или `LogAttrs()`:

```go
opts := &slog.HandlerOptions{
	Level: LevelTrace,
}
logger := slog.New(slog.NewJSONHandler(os.Stdout, opts))
ctx := context.Background()
logger.Log(ctx, LevelTrace, "Trace message")
logger.Log(ctx, LevelFatal, "Fatal level")
```