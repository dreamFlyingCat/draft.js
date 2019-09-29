# draft.js制作自己的富文本编辑器

Draft.js是一个框架，用于在React中构建富文本编辑器。

## 安装 

    npm install draft-js react react-dom
    # or alternately
    yarn add draft-js react react-dom

Draft.js使用了一些较新的ecmascript功能，所以IE11无法支持。create-react-app默认的babel配置不能完全转码Draft，使用过程中遇到了问题，尝试安装polyfill解决。

    npm install draft-js react react-dom babel-polyfill
    # or
    yarn add draft-js react react-dom es6-shim

## 使用

Editor是一个受控的ContentEditable组件，可以使用`RichUtils`和`Modifier`提供的方法修改文本或者样式。

    import React from 'react';
    import ReactDOM from 'react-dom';
    import {Editor, EditorState} from 'draft-js';

    class MyEditor extends React.Component {
    constructor(props) {
        super(props);
        this.state = {editorState: EditorState.createEmpty()};
        this.onChange = (editorState) => this.setState({editorState});
    }
    render() {
        return (
            <Editor editorState={this.state.editorState} onChange={this.onChange} />
        );
    }
    }

    ReactDOM.render(
        <MyEditor />,
        document.getElementById('container')
    );
## Editor组件

可以接受以下参数

- editorState 必须，编辑器完整状态的不可变快照
- onChange 必须，用于更新编辑器内容，会创建一个新的`editorState`
- placeholder  可选，当编辑器为空时显示的占位符字符串，可以使用CSS设置样式
- readOnly 可选，设置只读
- textAlignment 可选，设置文本的对齐方式
- textDirectionality 可选，设置文本的方向
- blockStyleFn 可选，给不同类型的block添加不同的class
- blockRendererFn 可选，自定义块渲染，将特定的文本渲染为自定义组件
- blockRendererMap 可选，block渲染方式的map图
- customStyleFn 可选，内联样式转换为应用于文本范围的CSS对象
- customStyleMap 可选，customStyleFn的map图
- autoCapitalize 可选，是否自动大写转换
- autoComplete 可选，否打开自动完成功能
- autoCorrect 可选，否打开自动修正功能
- spellCheck 可选，是否打开拼写检查
- stripPastedStyles 可选，是否从粘贴的内容中删除除明文以外的所有信息
- tabIndex 可选，指定tab键顺序
- editorKey 可选，服务器端使用Editor组件时使用
- handleReturn 可选，处理`RETURN`键按下事件
- handleKeyCommand 可选，处理内置的编辑器键盘事件
- handleBeforeInput 可选，输入前触发的事件，用法示例：用户键入-在block的开头，可以将该ContentBlock转换为无序列表项。
- handlePastedText 可选，处理直接粘贴到编辑器中的文本和html
- handlePastedFiles 可选，处理直接粘贴到编辑器中的文件
- handleDroppedFiles 可选，处理已放入编辑器的文件
- handleDrop 可选，处理其他放置操作
- keyBindingFn 可选，自定义按键绑定的处理事件
- onFocus 可选，获取焦点触发的事件
- onBlur 可选，失去焦点触发的事件
- focus 可选，获取焦点
- blur 可选，失去焦点

## EditorChangeType

`EditorChangeType`是一个枚举，它列出了可以被Draft模型处理的改变操作集。它被展示成流类型，作为字符串的并集。
`EditorChangeType`作为参数传递给`EditorState.push`，并且表明了正在执行的转换到新的`ContentState`这一改变操作的类型，执行完毕后会返回新的`ContentState`。

常用的值：

- insert-characters 在选择位置插入一个或多个字符
- delete-character 向前删除一个字符
- backspace-character 向后删除一个字符 
- change-block-data 改变ContentBlock对象
- change-block-type 改变ContentBlock的类型
- change-inline-style 改变选中的文本的样式
- insert-fragment 在选择状态下插入内容的“片段”
- adjust-depth 改变`ContentBlock`对象的深度
- apply-entity 使用entity
- redo 执行重做
- undo 执行撤销
- remove-range 删除多个字符或块
- spellcheck-change 执行拼写检查或自动更正更改

示例

        const selectionState = editorState.getSelection()
        const contentStateWithAtomic = Modifier.insertText(contentState, selectionState, text + '', null, entityKey)
        const newEditorState = EditorState.push(editorState, contentStateWithAtomic, 'insert-characters')
        this.setState({
            editorState: newEditorState,
        })

## EditorState

 `EditorState`是编辑器顶级的state对象，是编辑器当前完整状态的快照，是不可变的，包含以下信息：

- 文本信息
- 选择（当前鼠标位置或者选中的文本）的状态
- contentState中的decorated
- 撤销/重做的堆栈
- content的最新更改

> 使用EditorState对象时，不应使用Immutable API。


### 实例方法

- getCurrentContent 返回当前的content
- getSelection 返回选择状态
- getCurrentInlineStyle 返回当前编辑器的内联样式
- getBlockTree 返回decorated和样式范围的不可变列表，在渲染时此对象用于将内容分解为适当的块

示例

        const editorState = this.state.editorState 
        const selectionState = editorState.getSelection() 
        const contentState = editorState.getCurrentContent()
        const contentStateWithEntity = contentState.createEntity('var', 'MUTABLE', { speed })
        const entityKey = contentStateWithEntity.getLastCreatedEntityKey()
        const contentStateWithAtomic = Modifier.applyEntity(
            contentState,
            selectionState,
            entityKey
        )
        const newEditorState = EditorState.push(editorState, contentStateWithAtomic, 'insert-characters')
        this.setState({
            speed,
            editorState: newEditorState,
        })


### 静态方法

- createEmpty 创建`EditorState`，`decorator`可以作为入参，返回带有默认配置的`EditorState`
- createWithContent `decorator`和`ContentState`入参，创建带有默认内容和默认配置的`EditorState`
- push 给`EditorState`传入指定的`ContentState`和`changeType`，返回新的`EditorState`
- create  传入自定义的配置对象，返回`EditorState`
- setInlineStyleOverride `DraftInlineStyle`入参，样式作用于将要插入的文本，返回新的`EditorState`，例如从光标位置应用粗体可以使用该接口
- acceptSelection 接受`SelectionState`为参数，返回新的`EditorState`，但是不展示选择状态
- forceSelection 接受`SelectionState`为参数，返回新的`EditorState`，强制展示选择状态
- moveSelectionToEnd 移动选择状态到编辑器的尾部，不强制聚焦
- moveFocusToEnd 移动选择状态到编辑器的尾部，强制聚焦
- undo 将撤销堆栈的顶部`ContentState`作为新的`ContentState`返回，实现撤销操作
- redo 将重做堆栈的顶部`ContentState`作为新的`ContentState`返回，实现重做操作
- set 传入设置，返回新的`EditorState`

示例

        let editorState
        const content = this.props.content
        if (content) {
            const contentState = convertFromRaw(content)
            editorState = EditorState.createWithContent(contentState, decorator)
        } else {
            editorState = EditorState.createEmpty(decorator);
        }
     
       this.setState({editorState: editorState})

### Properties和Getters

在多数情况下，上述实例和静态方法应足以管理你Draft编辑器的状态。以下是由EditorState追踪属性的完整列表，以及他们相应的getter方法。

>注意
>
> 使用EditorState对象的时候，不应该使用Immutable API，应该使用相应的getters和静态方法。

- allowUndo 是否允许编辑器进行撤销/重做行为，默认值是true； 使用`getAllowUndo()`获取。因为撤销/重做栈是内存保留的主要来源，如果你的编辑器交互不需要撤销/重做栈，你可能要考虑将这个值设置为false。
- currentContent 当前渲染的ContentState
- decorator 当前的decorator对象，`getDecorator()`获取
- selection 当前渲染的`SelctionState`
- directionMap block和文本的map，`getDirectionMap()`获取
- treeMap 编辑器中渲染的完整的`decorator`和样式树
- forceSelection 是否强制渲染当前的`SelectionState`
- inlineStyleOverride 下一个插入字符应用的内联样式值
- lastChangeType `ContentState`最后发生内容变化的类型

示例

        // 当编辑器文本内容改变时候...

        const shouldFilterPaste =
        editorState.getCurrentContent() !== oldState.getCurrentContent() &&
        editorState.getLastChangeType() === "insert-fragment"

        if(shouldFilterPaste){
            // ...
        }

## ContentState

`ContentState`是一个不可变记录，它代表以下各项的完整状态：

-  编辑器的完整contents：文本，块，内联样式和entity范围
- 编辑器的selection states：在渲染这些内容之前和之后

`ContentState`对象最常见的用法是通过EditorState.getCurrentContent()获取，该方法提供当前编辑器渲染的`ContentState`对象。使用`EditorState`对象来维护由`ContentState`对象组成的撤销与重做堆栈。


### 静态方法

- createFromText 传入字符串创建`ContentState`对象，并使用分隔符将字符串拆分为ContentBlock对象。如果没有显性提供分隔符，则使用'\n'分割。

        EditorState.set(editorState, {
            currentContent: ContentState.createFromText("")
        })

- createFromBlockArray 传入ContentBlock对象数组创建ContentState对象。默认的selectionBefore和selectionAfter状态将光标放在内容的起始位置。

### 实例方法

- getEntityMap 返回一个包含所有已创建的DraftEntity记录的存储对象。
- getBlockMap 返回表示整个文档状态的ContentBlock对象组成的完整有序映射。

            // 获取所有的颜色信息
            const blockMap = editorState.getCurrentContent().getBlockMap()
            const COLORS = []
            blockMap.map(function (block) {
                block.getCharacterList().map(function (char) {
                 char.getStyle().map(function (type) {
                    if(/^E_COLOR_#/.test(type)) COLORS.includes(type) || COLORS.push(type)
                  })
                })
            })

- getBlockForKey 返回与给定key对应的ContentBlock对象。

        const selection = editorState.getSelection()
        const contentState = editorState.getCurrentContent()
        const currentBlock = contentState.getBlockForKey(selection.getAnchorKey()) // 获取焦点位置
        const entityKey = currentBlock.getEntityAt(selection.getFocusOffset())
        const entity = entityKey && contentState.getEntity(entityKey)

- createEntity 返回新的`ContentState`对象，它的EntityMap中包含了新建的DraftEntity对象。调用getLastCreatedEntityKey可以获得新建的DraftEntity对象的字符串key。
- getLastCreatedEntityKey 返回最近一次创建的DraftEntityRecord对象的应用key。

        const selectionState = editorState.getSelection()
        const contentState = editorState.getCurrentContent()
        const contentStateWithEntity = contentState.createEntity('var', 'MUTABLE', { speed })
        const entityKey = contentStateWithEntity.getLastCreatedEntityKey()
        const contentStateWithAtomic = Modifier.applyEntity(
            contentState,
            selectionState,
            entityKey
        )

- getEntity 返回指定key对应的DraftEntityInstance对象。如果找不到与指定key对应的对象则抛出异常。
- mergeEntityData 由于`DraftEntityInstance`对象是不可变的，你不能使用传统的修改方式来修改它们的元数据。
- replaceEntityData replaceData方法与mergeData方法类似，除了replaceData方法会使用新数据替换全部旧数据。
- addEntity 向`contentState`添加`entity`，当`entity`已经被创建，并且需要添加到实体存储对象上时，add方法非常有用
- getPlainText 返回使用分隔符(delimiter)连接的完整的纯文本内容。如果没有指定分隔符，默认使用换行符(\u00A)连接。
- hasText 返回编辑内容是否包含文本
- getSelectionBefore 返回渲染`blockMap`之前编辑器中显示的`SelectionState`；当你在编辑器中执行undo操作时，当前ContentState中的selectionBefore将用来在合适的位置设置选区。
- getSelectionAfter 当执行任何导致`blockMap`渲染的操作时，选区位置将被设置在selectionAfter的位置。
- getKeyBefore 返回`blockMap`中指定key的前一个key，如果该key已经是第一个则返回null。
- getKeyBefore 返回blockMap中指定key的后一个key，如果该key已经是最后一个则返回null。
- getBlockBefore 返回blockMap中指定key的前一个`ContentBlock`对象，如果该key已经是第一个则返回null。
- getBlockAfter 返回blockMap中指定key的后一个`ContentBlock`对象，如果该key已经是最后一个则返回null。
- getBlocksAsArray 以数组形式返回blockMap中的所有`ContentBlock`。
- getFirstBlock 返回第一个`ContentBlock`对象。
- getLastBlock 返回最后一个`ContentBlock`对象。

## ContentBlock

`ContentBlock`是一个不可变的Record对象，表示编辑器内容中单个块的完整状态，包括以下信息：

- 文本内容
- 类型，比如段落、头部，列表项等
- entity、内联样式、深度信息等

`ContentState`对象包含`ContentBlock`对象的OrderedMap，它们一起构成了编辑器的全部内容。`ContentBlock`对象很类似于块级HTML元素，例如段落和列表。可用的类型有unstyled、paragraph、header-one、header-two、header-three、header-four、header-five、header-six、unordered-list-item、ordered-list-item、blockquote、code-block、atomic。可以使用构造函数直接创建新的`ContentBlock`对象。预期的Record值详情如下所述。

### styled和entity

`characterList`字段是一个不可变的`List`对象，包含块中所有字符的`CharacterMetadata`对象。我们通过这种方式用代码来构建块的样式和实体。编辑`ContentBlock`的函数可以对单个`List`对象执行截取、拼接以及其他`List`支持的操作。

示例

            // 获取所有的颜色信息
            const blockMap = editorState.getCurrentContent().getBlockMap()
            const COLORS = []
            blockMap.map(function (block) {
                block.getCharacterList().map(function (char) {
                 char.getStyle().map(function (type) {
                    if(/^E_COLOR_#/.test(type)) COLORS.includes(type) || COLORS.push(type)
                  })
                })
            })

### 实例方法

- getKey() 返回该`ContentBlock`的类型key。`ContentBlock`的key是由字母和数字组成的字符串。建议使用generateRandomKey方法来创建块的key。
- getType() 返回该`ContentBlock`的类型。大多数类型值与HTML块级元素一致。
- getText() 返回该`ContentBlock`中的全部纯文本内容。
- getCharacterList() 返回一个不可变的List对象，该List由`ContentBlock`中的每个字符的`CharacterMetadata`对象组成。
- getLength() 返回该`ContentBlock`中纯文本类容的长度。
- getDepth() 如果有，则返回当前块的深度值。当前只适用于列表项。
- getInlineStyleAt() 返回该`ContentBlock`中给定位置的`DraftInlineStyle`值。
- getEntityAt() 返回该`ContentBlock`中给定位置的实体key的值（如果没有则返回null）。
- getData() 返回块级元数据。
- findStyleRanges() 为该`ContentBlock`中每个连续的样式范围执行回调函数。
- findEntityRanges() 为该`ContentBlock`中每个连续的实体范围执行回调函数。

示例：

            // 获取所有的颜色信息
            const blockMap = editorState.getCurrentContent().getBlockMap()
            const COLORS = []
            blockMap.map(function (block) {
                block.getCharacterList().map(function (char) {
                 char.getStyle().map(function (type) {
                    if(/^E_COLOR_#/.test(type)) COLORS.includes(type) || COLORS.push(type)
                  })
                })
            })

## CharacterMetadata

`CharacterMetadata`是一个包含单一字符行内样式和实体信息的不可更改的Record对象。  
`CharacterMetadata`对象被即时的汇总和共享，如果两个字符拥有相同的行内样式和`entity`，会被视为相同的`CharacterMetadata`对象。因此，我们只需要尽可能多的组合带有`entity` key的内联样式合集，尽可能的占用更小的内存。 

### 静态方法

在底层，这些方法将利用对象池返回匹配到的对象，如果对象不存在则放回一个新的对象，因此最好使用静态方法创建和更新`CharacterMetadata`确保最大限度的复用。  
- create 根据提供的配置信息生成一个`CharacterMetadata`对象。
- applyStyle 在`CharacterMetadata`对象上应用一个(组)内联样式
- removeStyle 从`CharacterMetadata`对象中移除一个(组)内联样式。
- applyEntity 在`CharacterMetadata`对象上应用一个实体key，如果`entityKey`为null，则为移除实体key。

### 实例方法

- getStyle 返回字符的`DraftInlineStyle`对象
- hasStyle 返回该字符串是否包含指定的样式
- getEntity 返回应用到该字符的实体key

使用方法可以参考`删除字符上指定的样式`示例

## Entities

entity是一个对象，用于特殊情况，给一些文本上附加一些额外的信息，它有3个属性：
 
- type entity的类型，例如`LINK`、`VAR`
- mutability 注释的文本是否可以编辑， Immutable只能删除不能编辑，Mutable可以编辑可以删除
- data entity的数据

所有的实体都存在`ContentState`中，ContentState中使用key引用entity，React组件用于装饰带注释的文本。

### 创建

使用`contentState.createEntity`创建entity，返回新的`ContentState`包括新创建的entity，可以使用`contentState.getLastCreatedEntityKey`获取最新创建的entity

    const contentState = editorState.getCurrentContent(); // 获取当前状态
    const contentStateWithEntity = contentState.createEntity(
    'LINK',
    'MUTABLE',
    {url: 'http://www.zombo.com'}
    );
    const entityKey = contentStateWithEntity.getLastCreatedEntityKey();
    // contentStateWithEntity新的contentState, selectionState
    const contentStateWithLink = Modifier.applyEntity(
    contentStateWithEntity,
    selectionState, 
    entityKey
    );

    const newEditorState = EditorState.push(editorState, { currentContent: contentStateWithLink });


### 检索

使用`getEntityAt`，参数为选中文本偏移值获取文本关联的entity的key，进一步获取entity存储的数据

    const contentState = editorState.getCurrentContent();
    const blockWithLinkAtBeginning = contentState.getBlockForKey('...');
    const linkKey = blockWithLinkAtBeginning.getEntityAt(0);
    const linkInstance = contentState.getEntity(linkKey);
    const {url} = linkInstance.getData();

### 修改

由于DraftEntityInstance记录是不可变的，因此不能直接修改entity的`data`属性。可以使用`mergeData`和`replaceData`两种Entity方法来修改entity。`mergeData`允许通过传入要合并的对象来更新数据，`replaceData`直接完全替换data。

## SelectionState

`SelectionState`是一个不可变(Immutable)记录(Record)，表示编辑器中的选择范围。  
经常通过`EditorState.getSelection()`来获得编辑器中当前的SelectionState对象。

### Keys 和 Offsets

一个选区内有两个重要的点：锚点和焦点。

> 锚点：使用鼠标进行选择时，锚点是文档中最初按下鼠标按钮的位置。用户使用鼠标或键盘更改选择时，锚点不会移动。  
> 焦点：选择的焦点是选择的终点。 使用鼠标进行选择时，焦点是在文档中释放鼠标按钮的位置。 当用户使用鼠标或键盘更改选择时，焦点就是移动的选择的结尾。  
> selection更多信息可以查看[WEB API - Selection](https://developer.mozilla.org/en-US/docs/Web/API/Selection#Glossary)

### Start/End vs. Anchor/Focus

锚点和焦点允许我们根据需要选择向前或向后选择，因此管理选择行为时，我们建议使用锚点和焦点来维护选择方向；在管理内容操作时，我们更建议使用开始和结束值。

例如，当我们基于SelectionState从块中截取一段文本时，是否向后选择内容并不重要：

        var selectionState = editorState.getSelection();
        var anchorKey = selectionState.getAnchorKey();
        var currentContent = editorState.getCurrentContent();
        var currentContentBlock = currentContent.getBlockForKey(anchorKey);
        var start = selectionState.getStartOffset();
        var end = selectionState.getEndOffset();
        var selectedText = currentContentBlock.getText().slice(start, end);

> 注意，对于SelectionState本身只会记录锚点和焦点值，开始和结束值需要计算才能得到。

### 静态方法

- createEmpty() 在指定的contentBock的零偏移处创建一个`SelectionState`对象，并将`hasFocus`设置为false。

示例

        var Immutable = require('immutable');
        var OrderedSet = Immutable.OrderedSet,
            Stack = Immutable.Stack;

        var firstKey = contentState.getBlockMap().first().getKey();
        EditorState.create({
        currentContent: contentState,
        undoStack: Stack(),
        redoStack: Stack(),
        decorator: decorator || null,
        selection: SelectionState.createEmpty(firstKey)
        });

### 实例方法

- getAnchorKey() 返回选区锚点位置的block key。
- getAnchorOffset() 返回选区锚点在所在content中的偏移。
- getStartKey() 返回选区起始位置的block key。
- getStartOffset() 返回选区的开始在所在content中的偏移。
- getEndKey() 返回包含选区结束位置的block key。
- getEndOffset() 返回选区的结束在所在content中的偏移。
- getFocusKey() 返回选区焦点位置的block key。
- getFocusOffset() 返回选区焦点位置在所在content中的偏移。
- getIsBackward() 返回焦点位置是否在锚点位置前面。
- getHasFocus() 返回编辑器是否聚焦。
- isCollapsed() 返回选区是否处于折叠状态。
- hasEdgeWithin() 返回选区范围是否与指定的块内指定的开始/结束位置发生重叠。
- serialize() 返回`SelectionState`的一个序列化版本，调试时非常有用。


        var startKey = selection.getStartKey();
        var startOffset = selection.getStartOffset();
        var startBlock = content.getBlockForKey(startKey);

        // 如果光标不在block的开头，往后找
        if (startOffset > 0) {
            return startBlock.getInlineStyleAt(startOffset - 1);
        }
        if (startBlock.getLength()) {
            return startBlock.getInlineStyleAt(0);
        }

### 属性

- anchorKey 包含选择范围锚点的block key
- anchorOffset 选择范围锚点位置的偏移量
- focusKey 选择范围焦点的block key
- focusOffset 选择范围焦点位置的偏移量
- isBackward 锚点位置是否在焦点位置后面
- hasFocus 编辑器是否聚焦

        var selectionState = SelectionState.createEmpty('foo');
        var updatedSelection = selectionState.merge({
        focusKey: 'bar',
        focusOffset: 0,
        });
        var anchorKey = updatedSelection.getAnchorKey(); // 'foo'
        var focusKey = updatedSelection.getFocusKey(); // 'bar'

## RichUtils

RichUtils提供了一系列的静态方法来修改文本样式，这些方法接受EditorState作为参数，返回新的EditorState。下例中使用RichUtils的toggleInlineStyle方法给文本加粗。

### handleKeyCommand通过键盘设置文本样式

RichUtils支持一些常用的键盘操作，例如`Cmd+B` (bold)加粗, `Cmd+I` (italic)斜体等。
我们可以使用`handleKeyCommand`属性检测并处理键盘操作，并hook到RichUtils中删除或者使用所需的样式。

    import {Editor, EditorState, RichUtils} from 'draft-js';

    class MyEditor extends React.Component {
        constructor(props) {
            super(props);
            this.state = {editorState: EditorState.createEmpty()};
            this.onChange = (editorState) => this.setState({editorState});
            this.handleKeyCommand = this.handleKeyCommand.bind(this);
        }
        handleKeyCommand(command, editorState) {
            const newState = RichUtils.handleKeyCommand(editorState, command);
            if (newState) {
            this.onChange(newState);
            return 'handled';
            }
            return 'not-handled';
        }
        render() {
            return (
                <Editor
                    editorState={this.state.editorState}
                    handleKeyCommand={this.handleKeyCommand}
                    onChange={this.onChange}
                />
            );
        }
    }

> 提供给handleKeyCommand的命令是一个字符串值，从DOM键盘事件映射中获取。 editorState是最新的编辑器状态，处理键盘命令可能修改编辑器内部状态。 在handleKeyCommand函数中会使用此editorState的实例。 更多信息请查看[详细信息](https://draftjs.org/docs/advanced-topics-key-bindings.html)。

### 静态方法

- toggleInlineStyle 在选中区域上切换指定的内联样式

        editorState = RichUtils.toggleInlineStyle(editorState, 'BOLD')
- handleKeyCommand 根据输入的键盘命令，可能修改编辑器内部状态

其他静态方法请参考[详情](https://draftjs.org/docs/api-reference-rich-utils)

## Decorator

除了使用自定义样式外，我们也可以使用自定义组件来渲染特定的内容。为了支持自定义富文本的灵活性，Draft.js提供了一个decrator系统。Decorator基于扫描给定ContentBlock的内容，找到满足与定义的策略匹配的文本范围，然后使用指定的React组件呈现它们。

可以使用CompositeDecorator类定义所需的装饰器行为。 此类允许你提供多个DraftDecorator对象，并依次搜索每个策略的文本块。

Decrator 保存在EditorState记录中。当新建一个EditorState对象时，例如使用EditorState.createEmpty()，可以提供一个decorator。

下面的例子中，使用CompositeDecorator搜索所有type为`var`的entity，并自定义渲染的组件：

        const decorator = new CompositeDecorator([
            // 规定类型为var的entity显示方式
            {
                strategy: (contentBlock, callback, contentState) => {
                    contentBlock.findEntityRanges(
                        (character) => {
                            const entityKey = character.getEntity();
                            return (
                                entityKey !== null &&
                                contentState.getEntity(entityKey).getType() === 'var'
                            );
                        },
                        callback
                    );
                },
                component: ({ entityKey, contentState, children }) => {
                    const { speed } = contentState.getEntity(entityKey).getData()
                    return (
                        <var className="speed-text"
                            data-speed={speed} title={'语速：' + speed}
                        >{children}</var>
                    )
                },
            }
            // ...
        ])

        const editorState = EditorState.createEmpty(decorator);
        this.setState({editorState: editorState})

        // 修改Decorator

        function turnOffHandleDecorations(editorState) {
            const onlyHashtags = new CompositeDecorator([{
                strategy: hashtagStrategy,
                component: HashtagSpan,
            }]);
            return EditorState.set(editorState, {decorator: onlyHashtags});
        }

## Modifier

`Modifier`模块是一组实用的静态函数，主要封装`ContentState`对象上的各种常用编辑操作。任何情况下，这些方法都接收具有相关参数的`ContentState`对象，并返回一个新的`ContentState`对象。如果实际并未发生任何编辑行为，将原样返回输入的`ContentState`对象。

### 静态方法

- applyEntity  将entity应用于整个选中的范围，如果`entityKey`为null，则移除整个范围中的所有实体。
- applyInlineStyle 将指定的内联样式应用于整个选中的范围。
- removeInlineStyle 从整个选中范围中移除指定的内联样式。
- insertText  插入文本
- replaceText 使用提供的字符串替换该`ContentState`的指定范围的内容并将内联样式和entity key应用于整合插入的字符串。
- moveText 将指定范围(removal)移动到目标范围(target)，并替换目标范围的文本。
- replaceWithFragment 使用fragment替换目标范围(target)，我们将粘贴的内容转换为要插入到编辑器的片段，然后使用该方法来添加。
- removeRange 从编辑器中移除整个文本范围。删除方向(removalDirection)对于正确的实体删除行为非常重要。
- splitBlock 将选中的块拆分为两个块。仅供选区折叠时使用。
- mergeBlockData 更新所有选中块的数据。

## 常用实例

### 过滤指定字符

        function replaceTextBySpaces(characters /*: $ReadOnlyArray<string>*/, content /*: ContentState*/) {
            var blockMap = content.getBlockMap();

            var blocks = blockMap.map(function (block) {
                var text = block.getText();

                // Only replaces the character(s) with as many spaces as their length,
                // so that style and entity ranges are left undisturbed.
                // If we want to completely remove the character, we also need to filter
                // the corresponding CharacterMetadata entities.
                var newText = characters.reduce(function (txt, char) {
                return txt.replace(new RegExp(char, "g"), " ".repeat(char.length));
                }, text);

                return text !== newText ? block.set("text", newText) : block;
            });

            return content.merge({
                blockMap: blockMap.merge(blocks)
            });
        };

### 撤销操作

        function undo(editorState) {
            if (!editorState.getAllowUndo()) {
                return editorState;
            }

            var undoStack = editorState.getUndoStack();
            var newCurrentContent = undoStack.peek();
            if (!newCurrentContent) {
                return editorState;
            }

            var currentContent = editorState.getCurrentContent();
            var directionMap = EditorBidiService.getDirectionMap(newCurrentContent, editorState.getDirectionMap());

            return EditorState.set(editorState, {
                currentContent: newCurrentContent,
                directionMap: directionMap,
                undoStack: undoStack.shift(),
                redoStack: editorState.getRedoStack().push(currentContent),
                forceSelection: true,
                inlineStyleOverride: null,
                lastChangeType: 'undo',
                nativelyRenderedContent: null,
                selection: currentContent.getSelectionBefore()
            });
        };

### 创建entity并插入绑定的文本
        
       const time = this.state.waitMills

        const editorState = this.state.editorState
        const contentState = editorState.getCurrentContent()
        const contentStateWithEntity = contentState.createEntity('dfn', 'IMMUTABLE', { cls: 'wait', val: time })
        const entityKey = contentStateWithEntity.getLastCreatedEntityKey()
        const selectionState = editorState.getSelection()
        const contentStateWithAtomic = Modifier.insertText(contentState, selectionState, time + '', null, entityKey)
        const newEditorState = EditorState.push(editorState, contentStateWithAtomic, 'insert-characters')
        this.setState({
            editorState: newEditorState,
        })

### entity过滤和entity data过滤

        var entityTypes = [
            {
            type: "IMAGE",
            attributes: ["src"]
          },
          {
            type: "LINK",
            attributes: ["url"],
          }
        ]

        var content =  editorState.getCurrentContent()
        var newContent = content;
        var entities = {};

        // 获取content中所有的entity
        newContent.getBlockMap().forEach(function (block) {
            block.findEntityRanges(function (char) {
                var entityKey = char.getEntity();
                if (entityKey) {
                    var entity = newContent.getEntity(entityKey);
                    entities[entityKey] = entity;
                }
            });
        });

        Object.keys(entities).forEach(function (key) {
                var entity = entities[key];
                var data = entity.getData();
                // 获取entity对应的过滤规则
                var config = entityTypes.find(function (t) {
                    return t.type === entity.getType();
                });
                var whitelist = config ? config.attributes : null;

                // 白名单没有定义，保留所有的entity.
                if (!whitelist) {
                    return data;
                }

                var newData = whitelist.reduce(function (attrs, attr) {
                    // 过滤entity data的属性
                    
                    if (data.hasOwnProperty(attr)) {
                        attrs[attr] = data[attr];
                    }

                    return attrs;
            }, {});

            newContent = newContent.replaceEntityData(key, newData);
        });
### 克隆entity

        function cloneEntities(content /*: ContentState*/) {
            var newContent = content;
            var blockMap = newContent.getBlockMap();

            var encounteredEntities = [];

            // 标记需要克隆的范围，避免重复。
            var shouldCloneEntity = function shouldCloneEntity(firstChar) {
                var key = firstChar.getEntity();

                if (key) {
                    if (encounteredEntities.includes(key)) {
                        return true;
                    }

                    encounteredEntities.push(key);
                }

                return false;
            };

            // 更新包含与其他范围指向同一实体的范围的block

            var blocks = blockMap.map(function (block) {
                var newChars = block.getCharacterList();
                var altered = false;

                // 获取需要克隆的entity
                var updateRangeWithClone = function updateRangeWithClone(start, end) {
                    var key = newChars.get(start).getEntity();
                    var entity = newContent.getEntity(key);

                    newContent = newContent.createEntity(entity.getType(), entity.getMutability(), entity.getData());
                    var newKey = newContent.getLastCreatedEntityKey();

                    // Update all of the chars in the range with the new entity.
                    newChars = newChars.map(function (char, i) {
                        if (start <= i && i <= end) {
                        return draftJs.CharacterMetadata.applyEntity(char, newKey);
                        }

                        return char;
                    });

                    altered = true;
                };

                block.findEntityRanges(shouldCloneEntity, updateRangeWithClone);

                return altered ? block.set("characterList", newChars) : block;
            });

            return newContent.merge({
                blockMap: blockMap.merge(blocks)
            });
        };

### 给选中区域添加样式

        class MyEditor extends React.Component {
        // …

        _onBoldClick() {
            this.onChange(RichUtils.toggleInlineStyle(this.state.editorState, 'BOLD'));
        }

        render() {
            return (
                <div>
                    <button onClick={this._onBoldClick.bind(this)}>Bold</button>
                    <Editor
                    editorState={this.state.editorState}
                    handleKeyCommand={this.handleKeyCommand}
                    onChange={this.onChange}
                    />
                </div>
                );
            }
        }

### 删除字符上指定的样式

        var styes = ["BOLD", 'ITALIC']
        var content = editorstate.getCurrentContent()
        var blockMap = content.getBlockMap();

        var blocks = blockMap.map(function (block) {
            var altered = false;

            var chars = block.getCharacterList().map(function (char) {
                var newChar = char;

                char.getStyle().filter(function (type) {
                    return !styes.includes(type);
                }).forEach(function (type) {
                    altered = true;
                    newChar = CharacterMetadata.removeStyle(newChar, type);
                });

                return newChar;
            });

            return altered ? block.set("characterList", chars) : block;
        });

        return content.merge({
            blockMap: blockMap.merge(blocks)
        });

### 自定义组件渲染

将特定格式的内容渲染为指定的组件，

先自定义渲染函数和组件: 只要在Editor上加`blockRendererFn`，

        const ImgComponent = (props) => {
            return (
                <img
                    style={{height: '300px', width: 'auto'}}
                    src={props.blockProps.src}
                    alt="图片"/>
            )
        }

        function myBlockRenderer(contentBlock) {
            
            // 获取到contentBlock的文本信息，可以用contentBlock提供的其它方法获取到想要使用的信息
            const text = contentBlock.getText();

            // 我们假定这里图片的文本格式为![](htt://....)
            let matches = text.match(/\!\[(.*)\]\((http.*)\)/);
            if (matches) {
                return {
                    component: ImgComponent,  // 指定组件
                    editable: false,  // 这里设置自定义的组件可不可以编辑，因为是图片，这里选择不可编辑
                    // 这里的props在自定义的组件中需要用this.props.blockProps来访问
                    props: {
                        src: matches[2],,
                    }
                };
            }
        }

之后只要在Editor上加`blockRendererFn`:

        <Editor
            editorState={this.state.editorState}
            onChange={this.onChange}
            blockRendererFn={myBlockRenderer}/>

### 获取选区的entity data

        const editorState = this.state.editorState
        const selection = editorState.getSelection()
        const contentState = editorState.getCurrentContent()
        const currentBlock = contentState.getBlockForKey(selection.getAnchorKey())
        const entityKey = currentBlock.getEntityAt(selection.getFocusOffset())
        const entity = entityKey && contentState.getEntity(entityKey)
        return entity ? entity.getData() : {}

