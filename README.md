# Как оптимизировать производительность нашего React приложения?

## React DevTools Profiler

Если у вас возникли проблемы с 
производительностью одного из компонентов, 
следует начать с 
"профилировщика" инструментов разработчика React.

Profiler измеряет то, как часто рендерится 
React-приложение и какова «стоимость» этого. 
Его задача — помочь найти медленные части приложения, 
которые можно оптимизировать 
(например, через мемоизацию).

```javascript
render(
  <App>
    <Profiler id="Navigation" onRender={onRenderCallback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);

function onRenderCallback(
    id, // проп "id" из дерева компонента Profiler, для которого было зафиксировано изменение
    phase, // либо "mount" (если дерево было смонтировано), либо "update" (если дерево было повторно отрендерено)
    actualDuration, // время, затраченное на рендер зафиксированного обновления
    baseDuration, // предполагаемое время рендера всего поддерева без кеширования
    startTime, // когда React начал рендерить это обновление
    commitTime, // когда React зафиксировал это обновление
    interactions // Множество «взаимодействий» для данного обновления 
) {
    // Обработка или логирование результатов...
}
```

## Использование метода shouldComponentUpdate() или React.memo()

Нужно понимать, что при рендере родителя 
ререндерится все дерево чайлдов. 
Это можно избежать используя PureComponent или Memo.

Указанный метод позволяет нам 
контролировать процесс рендеринга. 
Предположим, что мы хотим предотвратить повторный рендеринг определенного компонента. 
Для этого мы просто возвращаем false из shouldComponentUpdate(). 
Как видно из примера реализации метода, мы можем сравнивать 
текущее и следующее состояние и пропсы для определения необходимости перерисовки:

```javascript
shouldComponentUpdate(nextProps, nextState){
  return nextProps.id !== this.props.id
}
```

Если функциональный компонент рендерит одинаковый результат для одних и тех 
же пропов,
вы можете обернуть его в React.memo() для повышения производительности в
некоторых случаях посредством сохранения (запоминания, мемоизации) результата. 
Это означает, что React пропустит рендеринг компонента и
воспользуется последним результатом рендеринга.

React.memo() проверяет только изменение props. 
Если в вашем компоненте используются хуки useState или useContext, 
компонент будет повторно отрисовываться при изменении state или context.

## 5. "Виртуализация" длинных списков

Для решения проблем, связанных с длинной новостной лентой, команда 
React рекомендует использовать технику под названием windowing. 
Данная техника заключается в рендеринге только видимых пользователю в 
настоящий момент элементов списка (+/- определенный отступ), 
что повышает скорость рендеринга. При прокрутке страницы новые
элементы извлекаются и рендерятся. react-window и react-virtualized являются двумя 
популярными библиотеками для "виртуализации" длинных списков.

## Использование Throttle и Debounce

`Throttle`

Данная техника предотвращает вызов функции более одного раза 
за определенный промежуток времени. В примере ниже обработчик 
события нажатия кнопки срабатывает не чаще одного раза в секунду.

`Debounce`

Данная техника позволяет обеспечить вызов функции только по
истечении определенного времени с момента ее последнего вызова. 
Это может быть полезным, когда необходимо выполнять "дорогие"
вычисления в ответ на событие, которое 
возникает очень часто (например, событие прокрутки или нажатия клавиши).

## Устранение лишних узлов 

Фрагменты позволяют группировать списки потомков без добавления лишних узлов DOM:

```javascript
class App extends React.Component {
  render() {
    return (
      <React.Fragment>
        <ChildA />
        <ChildB />
        <ChildC />
      </React.Fragment>
    )
  }
}
```
