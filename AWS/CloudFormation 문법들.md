# CloudFormation 문법들

- SAM을 비롯하여 AWS 리소스를 CloudFormation으로 정의할 때, 잊지 않고 싶은 문법들을 계속해서 업데이트한다.

## !Equals

- example
    
    ```yaml
    Conditions:
      IsProd: !Equals [!Ref Environment, prod]
      IsUrgent: !Equals [!Ref Urgent, dev]
    ```
    
- `!Equals` 함수는 두 값이 동일한지 비교. 같으면 `true` 다르면 `false` 반환
- `IsProd` 조건: `Environment` 파라미터가 'prod'와 같으면 true
- `IsUrgent` 조건: `Urgent` 파라미터가 true와 같으면 true

## !If

- example
    
    ```yaml
    DeploymentPreference:
            Type: !If 
              - IsUrgent
              - AllAtOnce
              - !If [IsProd, Linear10PercentEvery1Minute, AllAtOnce]
    ```
    
- 만약에 리스트의 첫번째 요소가 true면 두번째 요소, 아니라면 세번째 요소의 값을 선택한다.
- 예시 flow
    - `IsUrgent`가 `true`이면 `DeploymentPreference`의 `Type`은 `AllAtOnce`
    - 아니라면 두번째 `!If`문을 평가하고
    - `IsProd`가 `true`라면 `DeploymentPreference`의 `Type`은 `Linear10PercentEvery1Minute`
    - `false`라면 `AllAtOnce`
- 수도코드로 표현하면 다음과 같다.

```jsx
if IsUrgent:
    DeploymentPreference.Type = "AllAtOnce"
else:
    if IsProd:
        DeploymentPreference.Type = "Linear10PercentEvery1Minute"
    else:
        DeploymentPreference.Type = "AllAtOnce"
```