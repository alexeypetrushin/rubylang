# Micon - убийца зависимостей и конфигов

[Micon][micon] позволяет быстро и бесшумно избавится от сложных зависимостей и конфигураций. Или, другими словами, он позволяет:

- Разделить приложение на части которые ничего не знают друг о друге кроме имени и нескольких публичных методов и полностью избавится от конфигов в коде.
- Просто перегружать любые значения конфигурации любых компонентов в различных вариантах сборки и енвайронментах.
- Полностью избавится от 'require' и вообще какого-либо упоминания зависимостей в виде путей и файлов.
- Автоматически управлять жизненным циклом компонентов.
- Автоматически собирать компоненты для тестов и обеспечить изоляцию между тестами (восстановение состояния компонента после теста).
- За все это практически не нужно ничем платить (кроме запоминания 2х методов и помещения определений компонетнов в папку lib/components)

Технически, [Micon][micon] это Dependency Injector, но из-за свой простоты и невидимости он выглядит словно чужой по сравнению со своими сложными и раздутыми IoC / DI коллегами.

[micon]: https://github.com/alexeypetrushin/micon
[icon]: https://firenet.s3.amazonaws.com/fs/4da14c38743f0f218d00000f/4da14ea9743f0f218d000015/4e4ae44860380b23a00001de/assassin.icon.png

Tags : DI, IoC, Код, Сложность