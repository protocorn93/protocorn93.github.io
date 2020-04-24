---
title: App Programming Guide for iOS (1)
subtitle: Introduction, Expected App Behaviors
date: 2018-08-22 15:22:49
tags: ["iOS", "Swift"]
category: ["Programming Guide"]
---

Xcode로부터 생성된 모든 프로젝트는 생성 그 즉시 시뮬레이터나 디바이스에서 실행이 가능합니다. 하지만 이렇게 단순히 실행이 된다는 이유 하나만으론 앱 스토에 배포될 준비가 되었다 할 수 없습니다. 모든 앱은 사용자에게 좋은 사용 경험을 보장할만한 어느 정도 수준의 **customization**이 필요합니다. 이러한 customization의 범위는 앱의 아이콘부터 앱의 설계 수준까지 다양합니다. 이번 챕터에선 모든 앱에서 처리해야하는 동작과 계획 과정 초기에 고려해야할 것들에 대해 설명합니다. 

------

훌륭한 사용자 경험 전달을 보장하기 위해서 앱은  iOS와 함께 동작해야 합니다. 훌륭한 사용자 경험이란 단지 디자인뿐만 아니라 더 많은 요소들을 함축적으로 내포합니다. 앱의 사용자는 앱이 빠른 속도로 반응하고 적은 전력만을 사용하기를 기대합니다. 앱은 모든 최신 iOS 기기를 지원해야하며 현재 사용되고 있는 기기에서 최적화된 것처럼 보여야 합니다. 

#### At a Glance

**Apps Are Expected to Support Key Features**

시스템은 모든 앱이 앱 아이콘이나 앱의 기능을 나타내는 정보와 같이 특정 리소스와 설정 데이터들을 갖고 있을 것이라고 간주합니다. Xcode를 통해 프로젝트를 생성한다면 몇몇의 정보는 기본적으로 주어지지만 추가적인 리소스 파일이나 앱의 기능의 명시는 앱이 배포되기 전에 반드시 설정해주어야 합니다.

- [Expected App Behaviors](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW2)

**Apps Follow Well-Defined Execution Paths**

사용자가 앱을 실행한 시점부터 종료하는 시점까지 앱은 잘 정의된 실행 단계(경로)를 따라야 합니다. 앱의 생명주기 동안 앱은 foreground와 background를 오갈 수 있고, 종료된 후 재실행될 수 있으며, 또한 잠시 수면 상태에 돌입할 수 있습니다. 이렇게 새로운 상태로 변환될 때 마다 앱의 상태에 대한 기대는 달라집니다. foreground 상태의 앱은 거의 모든 것을 할 수 있지만 background에선 최소한의 행위만을 해야 합니다. 개발자는 이러한 상태 변화를 활용하여 앱의 동작을 적절하게 조정할 수 있습니다. 

- [The App Life Cycle](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/TheAppLifeCycle/TheAppLifeCycle.html#//apple_ref/doc/uid/TP40007072-CH2-SW1), [Strategies for Handling App State Transitions](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforHandlingAppStateTransitions/StrategiesforHandlingAppStateTransitions.html#//apple_ref/doc/uid/TP40007072-CH8-SW1)

**App Must Run Efficiently in a Multitasking Enviornment**

사용자에게 베터리 Life는 성능, 반응 속도, 훌륭한 사용자 경험만큼 중요합니다. 앱의 베터리 사용량을 축소시킨다면 사용자는 해당 앱을 재충전 없이 하루종일 사용할 수 있겠지만, 빠른 앱 런칭 속도와 앱의 동작 속도또한 그만큼 중요합니다. iOS의 멀티태스킹 환경은 사용자가 기대하는 반응 속도나 사용자 경험을 희생시킬 필요 없이 좋은 베터리 Life를 제공합니다. 하지만 이를 위해선 앱이 시스템이 정의하고 제공하는 행위들을 수행해야 합니다. 

- [Background Execution](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW1), [Strategies for Handling App State Transitions](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforHandlingAppStateTransitions/StrategiesforHandlingAppStateTransitions.html#//apple_ref/doc/uid/TP40007072-CH8-SW1)

**Communication Between Apps Follows Specific Pathways**

보안성의 목적으로 iOS 앱들은 각각의 샌드박스에서 실행되며 다른 앱과의 제한된 상호작용만 허용됩니다. 시스템 상의 다른 앱들과 통신을 하고 싶다면 특별한 방법이 존재합니다. 

- [Inter-App Communication](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html#//apple_ref/doc/uid/TP40007072-CH6-SW2)

**Performance Tuning is Important for Apps**

앱에 의해 수행되는 모든 작업은 그에 맞는 에너지 비용이 따르기 마련입니다. 사용자의 베터리를 많이 잡아먹는 앱은 부정적인 사용자 경험을 제공하고 이는 앱의 삭제로 이어지기 됩니다. 그렇기 때문에 작업에 따른 비용을 항상 인지하고 있어야 하며 시스템이 제공하는 에너지 절전 기능을 활용해야 합니다.

- [Performance Tips](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/PerformanceTips/PerformanceTips.html#//apple_ref/doc/uid/TP40007072-CH7-SW1)

------

#### Providing the Required Resources

모든 앱은 디바이스 위에서 실행되기 위해선 다음 목록에 해당하는 리소스와 메타데이터들을 반드시 포함해야 합니다.

- **An information property-list file**

  - `Info.plist`는 시스템이 앱과 상호작용할 때 필요로 하는 메타데이터를 포함하고 있습니다. Xcode는 프로젝트의 생성 시 초기 설정에 맞게 자동으로 이 파일을 생성해줍니다. 이 파일은 제작하는 프로젝트의 특성에 따라 직접 수정할 수 있습니다.
  - [The Information Property List File](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW5)

- **A declaration of the app's required capabilites**

  - 모든 앱은 앱 동작에 필요한 하드웨어 기능들을 반드시 명시해주어야 합니다. 앱스토어는 이 정보를 이용하여 해당 앱이 사용자(앱스토에서 앱을 받으려는 사람)의 디바이스에서 동작할 수 있는지 없는지를 결정합니다. 역시 제작하는 프로젝트의 특성에 따라 직접 수정할 수 있습니다. 
  - [Declaring the Required Device Capabilities](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW4)

- **One or more icons**

  - 앱의 아이콘은 시스템에 의해 사용자의 홈 화면에서 확인할 수 있습니다. 또한 설정 앱에서 다른 버전의 앱 아이콘을 설정할 수 있습니다.
  - [App Icons](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW1)

- **One or more launch images**

  - 앱이 시작되고 앱의 UI가 준비될 때 까지 시스템은 임시로 launch image를 사용자에게 보여줍니다. launch image를 통해 사용자는 앱이 실행 중이고 곧 메인 화면으로 진입할 것이라는 것을 알 수 있습니다. 앱은 반드시 하나 이상의 launch image를 갖고 있어야 하며 다양한 시나리오에 맞춰 다양한 launch image를 사용할 수 있습니다.
  - [App Launch (Default) Images](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW3)


------

#### The App Bundle

iOS 앱을 빌드하면 Xcode 이를 **번들(bundle)**이라는 것으로 포장합니다. 번들은 하나의 디렉토리로 관련된 리소스들을 한 장소에 그룹지어 놓은 것을 의미합니다. iOS 앱 번들은 앱의 실행 파일과 앱 아이콘, 이미지 파일, 지역화된 컨텐츠와 같은 리소스 파일들을 포함하고 있습니다. 다음 테이블 표는 설명을 위한 전형적인 iOS 앱인 MyApp의 앱 번들에 포함되는 컨텐츠들을 설명합니다.

> 이 예제는 설명을 위한 목적으로 몇몇 파일은 여러분들의 앱 번들에 없을 수 있습니다. 



| File                                   | Example                  | Description                                                  |
| -------------------------------------- | ------------------------ | ------------------------------------------------------------ |
| App executable                         | `MyApp`                  | 실행 가능 파일은 앱의 컴파일 된 코드를 포함합니다. 실행 가능 파일의 이름은 앱의 이름에서 `.app` 확장자를 뺀 것과 동일합니다. **[필수]** |
| The information property list file     | `Info.plist`             | `Info.plist` 파일은 앱의 설정 정보를 포함하고 있으며 시스템은 이 정보를 사용하여 해당 앱과 어떻게 상호작용할지를 결정합니다. 이 파일명은 반드시 `Info.plist`이어야 합니다. **[필수]** |
| App icons                              | `Icon.png` `Icon@2x.png` | 홈 화면에서 앱을 나타내는 아이콘 **[필수]**                  |
| Launch Images                          | `Default.png`            | 앱의 실행 준비동안 사용자에게 보여지는 이미지로 앱이 UI를 보여줄 준비가 되면 사라지고 UI가 나타납니다. 최소한 한 개 이상의 이미지가 요구 됩니다. **[필수]** |
| Storyboard files (or nib files)        | `Main.storyboard`        | 스토리보드 파일은 앱이 디바이스 화면에 보여주는 뷰나 뷰 컨트롤러들을 포함합니다. 뷰 컨트롤러는 뷰들을 묶어 정렬하고 이들을 화면에 보여주는 역할을 합니다. 또한 스토리보드는 한 세트의 뷰들에서 다른 세트의 뷰들로의 전환 작업(**Segue**)에도 관여합니다. 메인 스토리보드 파일의 이름은 Xcode에 의해 프로젝트 생성시 자동으로 결정되는데 `Info.plist`에서 이를 변경해줄 수 있습니다. nib 파일을 사용하는 프로젝트에서도 마찬가지 입니다. 스토리보드 파일의 사용은 필수는 아니지만 권장됩니다. |
| Settings bundle                        | ` Settings.bundle`       | 만일 디바이스의 설정 앱에서 앱의 설정이 보여지길 원한다면 설정 번들을 반드시 포함시켜야 합니다. 이 번들은 선택 사항입니다. |
| Nonlocalized resource files            | `sun.png`                | 이미지, 사운드, 동영상과 같이 지역화되지 않은 파일이라 합니다. 이러한 모든 파일들은 앱 번들에 상단에 위치해야 합니다. |
| Subdirectories for localized resources | ` en.lproj` `fr.lproj`   | 지역화된 리소스들은 반드시 `.lproj` 접미사를 갖는 언어 별 프로젝트 디렉토리에 위치해야 합니다. iOS 앱은 국제화되어야 하며 지원하는 언어 별 `.lproj` 디렉토리를 갖고 있어야 합니다. |

> iOS 앱은 `Resource`라는 이름의 사용자 정의 디렉토리는 생성할 수 없습니다.

------

#### Supporting User Privacy

사용자의 개인정보를 다루기 위한 설계는 매우 중요합니다. 대부분의 iOS 디바이스에는 사용자가 외부나 다른 앱에 노출시키고 싶지 않은 민감한 데이터들을 갖고 있습니다. 만일 앱이 사용자 정보에 부적절한 방식으로 접근하거나 사용한다면 사용자는 해당 앱을 그 즉시 삭제할 것입니다.

이러한 사용자나 디바이스의 정보에 접근 사용자의 승낙하에 이루어져야 합니다. 게다가 사용자와 디바이스의 정보를 보호하는데 적당한 단계를 거쳐야 하며 데이터의 사용은 투명하게 이루어져야 합니다. 다음은 이러한 민간한 데이터에 접근할 때 취할 수 있는 행동의 좋은 예제들입니다. 

- 정부나 산업계의 가이드라인을 따른다.
- iOS 시스템 허가 설정에 의해 보호되고 있는 민감한 데이터에 접근 허가 요청은 앱이 해당 데이터를 필요로 하는 그 순간에 이루어져야 합니다. 이렇게 민감한 데이터에 접근을 요청할 때는 왜 필요하며 어떻게 사용할지에 대한 적절한 문구를 `Info.plist`에 해당 데이터 사용 요청 권한 문구에 명시해주어야 합니다. iOS 시스템 허가 설정에 해당하는 데이터는 위치, 연락처, 캘린더, 추억, 사진, 미디어 등이 있습니다. 사용자가 데이터의 접근을 허가하지 않는 상황에서도 합당한 대체 행위를 제공해야 합니다. 
- 데이터가 어떻게 사용되는지에 대해 투명하게 공개되어야 합니다. 예를 들어 앱스토에 앱을 배포할 때 데이터 사용 정책에 대한 URL을 메타데이터에 포함시킬 수 있습니다. 또한 앱의 설명에서 이러한 민감한 데이터 사용 정책에 대해 기술할 수도 있습니다. 
- 사용자나 디바이스에 정보에 대한 권한은 전적으로 사용자에게 있어야 합니다. 사용자가 언제든지 데이터 접근에 대해 거부할 수 있어야 합니다. 
- 작업을 수행하기 위한 최소한의 데이터만을 요청하고 사용해야 합니다. 불필요하거나 추후에 사용할 것 같다는 이유와 같이 명백한 이유 없는 불필요한 데이터의 요청이나 수집은 하지 말아야 합니다. 
- 앱에서 수집하려는 사용자나 디바이스 데이터는 적절한 단계를 거쳐 보호되어야 합니다. 앱 자체에서 이러한 정보들을 저장하려면 iOS의 데이터 보호 기능들을 사용하여 암호화된 데이터들을 저장해야 합니다. 사용자나 디바이스의 정보를 네트워크를 통해 전송하기 위해서는 App Transport Security를 사용해야 합니다. 
- 앱이 오디오 입력을 지원하는 경우 실제로 녹음을 시작하는 시점에 녹음을 위한 오디오 세션을 구성해야 합니다. 녹음을 시작할 때 사용자에게 이를 알리고 녹음을 중지할 수 있는 옵션도 제공해야 합니다.

------

참고자료

1. [App Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072-CH1-SW1)