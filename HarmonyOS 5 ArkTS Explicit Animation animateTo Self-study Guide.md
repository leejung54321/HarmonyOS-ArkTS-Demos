# HarmonyOS 5: ArkTS Explicit Animation animateTo Self-study Guide

In recent project development, I frequently needed to add transition animation effects to interface elements to enhance the user experience. During this process, I came across the `animateTo` global explicit animation interface provided by ArkTS. It offers a convenient way to insert transition effects for state changes caused by closure code, enabling smooth animations for layout width/height changes and content presentation. However, this interface has many usage details and related parameters, and although the documentation is detailed, it is fragmented, making it difficult for beginners to understand and master. Therefore, I decided to write this blog to organize my learning experience and understanding into a self-study guide, hoping to help other developers quickly and deeply master the use of the `animateTo` interface.

## I. Overview of the `animateTo` Interface

The `animateTo` interface provides an explicit way to add transition animations for state changes. It supports property animations, layout width/height change animations, etc. It should be noted that by default, content (such as text and Canvas content) will directly reach the end state. To make the content follow width/height changes, you can configure it using the `renderFit` property.

### Supported Versions

- Supported from API Version 7. For new content in subsequent versions, the starting version of the content is marked separately with a superscript.
- From API version 9, this interface supports use in ArkTS cards.
- From API version 10, you can use `animateTo` in `UIContext` to clarify the execution context of the UI.
- From API version 11, this interface supports use in meta services.

### Interface Definition

```typescript
animateTo(value: AnimateParam, event: () => void): void
```

- **Parameter Description**:
  - `value`: Type `AnimateParam`, required, used to set animation effect-related parameters.
  - `event`: Type `() => void`, required, specifies the closure function of the effect. The system will automatically insert transition animations for state changes in the closure function.

## II. Detailed Explanation of the `AnimateParam` Object

The `AnimateParam` object contains multiple parameters for configuring animation effects. Below is a detailed introduction to each parameter:

### 1. `duration`

- **Type**: `number`
- **Required**: No
- **Description**: Animation duration in milliseconds. The default value is 1000.
  - The maximum animation duration on ArkTS cards is 1000 milliseconds; if exceeded, it is fixed at 1000 milliseconds.
  - Values less than 0 are treated as 0.
  - Floating-point values are rounded down. For example, setting 1.2 is treated as 1.

### 2. `tempo`

- **Type**: `number`
- **Required**: No
- **Description**: Animation playback speed. The larger the value, the faster the animation; the smaller the value, the slower the animation. A value of 0 means no animation effect. The default value is 1.0. Values less than 0 are treated as 1.

### 3. `curve`

- **Type**: `Curve | ICurve9+ | string`
- **Required**: No
- **Description**: Animation curve. The default value is `Curve.EaseInOut`.

### 4. `delay`

- **Type**: `number`
- **Required**: No
- **Description**: Animation delay playback time in ms (milliseconds), with no delay by default. The default value is 0, and the value range is `(-∞, +∞)`.
  - `delay >= 0` means delayed playback, and `delay < 0` means early playback. For `delay < 0`:
    - When the absolute value of `delay` is less than the actual animation duration, the animation will directly move to the state at the moment of the absolute value of `delay` in the first frame after starting.
    - When the absolute value of `delay` is greater than or equal to the actual animation duration, the animation will directly move to the end state in the first frame after starting. The actual animation duration equals the single animation duration multiplied by the number of animation playbacks.
  - Floating-point values are rounded down. For example, setting 1.2 is treated as 1.

### 5. `iterations`

- **Type**: `number`
- **Required**: No
- **Description**: Number of animation playbacks. By default, it plays once. Setting to -1 means infinite playback. Setting to 0 means no animation effect. The default value is 1, and the value range is `[-1, +∞)`.

### 6. `playMode`

- **Type**: `PlayMode`
- **Required**: No
- **Description**: Animation playback mode. By default, it plays from the beginning after completion. The default value is `PlayMode.Normal`.
  - It is recommended to use `PlayMode.Normal` and `PlayMode.Alternate`, where the first round of the animation plays forward. If using `PlayMode.Reverse` and `PlayMode.AlternateReverse`, the first round of the animation plays in reverse, jumping to the end state at the beginning and then playing the animation in reverse.
  - When using `PlayMode.Alternate` or `PlayMode.AlternateReverse`, developers should ensure that the final animation state matches the state variable's value, i.e., the last round of the animation should play forward. When using `PlayMode.Alternate`, `iterations` should be an odd number. When using `PlayMode.AlternateReverse`, `iterations` should be an even number.
  - `PlayMode.Reverse` is not recommended, as it causes the animation to jump to the end state at the beginning and makes the final animation state differ from the state variable's value.

### 7. `onFinish`

- **Type**: `() => void`
- **Required**: No
- **Description**: Animation completion callback. When a UIAbility switches from the foreground to the background, finite loop animations still in progress will end immediately, triggering the completion callback.

### 8. `finishCallbackType11+`

- **Type**: `FinishCallbackType`
- **Required**: No
- **Description**: Defines the type of the `onFinish` callback in the animation. The default value is `FinishCallbackType.REMOVED`.
  - **`FinishCallbackType` Description**:
    - `REMOVED`: The callback is triggered when the entire animation ends and is immediately deleted.
    - `LOGICALLY`: The callback is triggered when the animation is logically in a descending state but may still be in its long-tail state.

### 9. `expectedFrameRateRange11+`

- **Type**: `ExpectedFrameRateRange`
- **Required**: No
- **Description**: Sets the expected frame rate for the animation.
  - **`ExpectedFrameRateRange` Description**:
    - `min`: Minimum expected frame rate.
    - `max`: Maximum expected frame rate.
    - `expected`: Optimal expected frame rate.

## III. Usage Notes

- Changing properties in an animation closure function with `duration` set to 0 can stop the property animation effect.
- If you need to create an animation when a component appears, you can implement the animation creation in `onAppear`. It is not recommended to call animations in `aboutToAppear` or `aboutToDisappear` because:
  - Calling an animation in `aboutToAppear` occurs before the `build` in the custom component is executed, and internal components have not been created. The animation timing is too early, and animation properties have no initial values, so the animation cannot affect the component.
  - When `aboutToDisappear` is executed, the component is about to be destroyed, and no animation can be done in `aboutToDisappear`.
- You can also use in-component transitions to animate when components appear and disappear. For properties not supported by in-component transitions, you can use `animateTo` to achieve component disappearance animation effects.

## IV. Example Code

### Example 1: Setting the Animation to Execute in `onAppear`

```typescript
@Entry
@Component
struct AnimateToExample {
  @State widthSize: number = 300
  @State heightSize: number = 120
  @State rotateAngle: number = 0
  private flag: boolean = true

  build() {
    Column() {
      Button('change size')
        .width(this.widthSize)
        .height(this.heightSize)
        .margin(40)
        .onClick(() => {
          if (this.flag) {
            animateTo({
              duration: 2500,
              curve: Curve.EaseIn,
              iterations: 4,
              playMode: PlayMode.Normal,
              onFinish: () => {
                console.info('play end')
              }
            }, () => {
              this.widthSize = 180
              this.heightSize = 80
            })
          } else {
            animateTo({}, () => {
              this.widthSize = 300
              this.heightSize = 120
            })
          }
          this.flag = !this.flag
        })
      Button('stop rotating')
        .margin(60)
        .rotate({ x: 0, y: 0, z: 1, angle: this.rotateAngle })
        .onAppear(() => {
          animateTo({
            duration: 1500,
            curve: Curve.EaseIn,
            delay: 600,
            iterations: -1,
            playMode: PlayMode.Alternate,
            expectedFrameRateRange: {
              min: 15,
              max: 130,
              expected: 70,
            }
          }, () => {
            this.rotateAngle = 120
          })
        })
        .onClick(() => {
          animateTo({ duration: 0 }, () => {
            this.rotateAngle = 0
          })
        })
    }.width('100%').margin({ top: 10 })
  }
}
```

### Example 2: Component Disappears After Animation Completion

```typescript
@Entry
@Component
struct AttrAnimationExample {
  @State heightSize: number = 120;
  @State isShow: boolean = true;
  @State count: number = 0;
  private isToBottom: boolean = true;

  build() {
    Column() {
      if (this.isShow) {
        Column()
          .width(220)
          .height(this.heightSize)
          .backgroundColor('green')
          .onClick(() => {
            animateTo({
              duration: 2200,
              curve: Curve.EaseInOut,
              iterations: 1,
              playMode: PlayMode.Normal,
              onFinish: () => {
                this.count--;
                if (this.count == 0 && !this.isToBottom) {
                  this.isShow = false;
                }
              }
            }, () => {
              this.count++;
              if (this.isToBottom) {
                this.heightSize = 70;
              } else {
                this.heightSize = 120;
              }
              this.isToBottom = !this.isToBottom;
            })
          })
      }
    }.width('100%').height('100%').margin({ top: 10 })
    .justifyContent(FlexAlign.End)
  }
}
```

Through the above introduction and examples, I believe you have a deeper understanding of the `animateTo` interface in ArkTS. In actual development, you can flexibly configure the parameters of the `AnimateParam` object according to specific needs to achieve various cool animation effects.