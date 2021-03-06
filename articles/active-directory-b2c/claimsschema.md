---
title: ClaimsSchema  - Azure Active Directory B2C | Microsoft Docs
description: 指定 Azure Active Directory B2C 中自訂原則的 ClaimsSchema 元素。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/05/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 4c3b3318e941723ec333597c7e4b3e48710152d1
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2020
ms.locfileid: "78397795"
---
# <a name="claimsschema"></a>ClaimsSchema

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

**ClaimsSchema** 元素會定義可當作原則一部分進行參考的宣告類型。 宣告結構描述是可在其中宣告您的宣告的位置。 宣告可以是名字、姓氏、顯示名稱、電話號碼及其他項目。 ClaimsSchema 元素包含 **ClaimType** 元素的清單。 **ClaimType** 元素包含 **Id** 屬性，此為宣告名稱。

```XML
<BuildingBlocks>
  <ClaimsSchema>
    <ClaimType Id="Id">
      <DisplayName>Surname</DisplayName>
      <DataType>string</DataType>
      <DefaultPartnerClaimTypes>
        <Protocol Name="OAuth2" PartnerClaimType="family_name" />
        <Protocol Name="OpenIdConnect" PartnerClaimType="family_name" />
        <Protocol Name="SAML2" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" />
      </DefaultPartnerClaimTypes>
      <UserHelpText>Your surname (also known as family name or last name).</UserHelpText>
      <UserInputType>TextBox</UserInputType>
```

## <a name="claimtype"></a>ClaimType

**ClaimType** 元素包含下列屬性：

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| Id | 是 | 要用於宣告類型的識別碼。 其他元素可以在原則中使用這個識別碼。 |

**ClaimType** 元素包含下列元素：

| 元素 | 發生次數 | 描述 |
| ------- | ----------- | ----------- |
| DisplayName | 1:1 | 要在各種畫面上顯示給使用者的標題。 此值可進行[當地語系化](localization.md)。 |
| DataType | 1:1 | 宣告的類型。 |
| DefaultPartnerClaimTypes | 0:1 | 要用於指定通訊協定的夥伴預設宣告類型。 此值可以使用 **InputClaim** 或 **OutputClaim** 元素中指定的 **PartnerClaimType** 來覆寫。 使用此元素來指定通訊協定的預設名稱。  |
| Mask | 0:1 | 遮罩字元的選擇性字串，可在顯示宣告時套用。 例如，可將電話號碼 324-232-4343 的遮罩設定為 XXX-XXX-4343。 |
| UserHelpText | 0:1 | 宣告類型的說明，有助於使用者了解其用途。 此值可進行[當地語系化](localization.md)。 |
| UserInputType | 0:1 | 輸入控制項的類型，應該在使用者手動輸入宣告類型的宣告資料時提供給他們使用。 請參閱本頁面稍後所定義的使用者輸入類型。 |
| 管理員説明文本 | 0:1 | 聲明類型的說明，有助於管理員瞭解其用途。 |
| 限制 | 0:1 | 此宣告的值限制，例如規則運算式 (Regex) 或可接受的值清單。 此值可進行[當地語系化](localization.md)。 |
PredicateValidationReference| 0:1 | 對 **PredicateValidationsInput** 元素的參考。 **PredicateValidationReference** 元素可讓您執行驗證程序來確定只會輸入正確格式的資料。 如需詳細資訊，請參閱[述詞](predicates.md)。 |



### <a name="datatype"></a>DataType

**DataType**元素支援以下值：

| 類型 | 描述 |
| ------- | ----------- |
|boolean|代表布林值 (`true` 或 `false`)。|
|date| 表示時間的瞬間，通常表示為一天的日期。 日期的值遵循 ISO 8601 約定。|
|dateTime|表示時間的瞬間，通常以一天的日期和時間表示。 日期的值遵循 ISO 8601 約定。|
|duration|表示以年、月、天、小時、分鐘和秒為單位的時間間隔。 的格式為`PnYnMnDTnHnMnS`，其中`P`表示為正值或`N`負值。 `nY`是後跟字`Y`的年數。 `nMo`是以字面`Mo`後跟的月份數。 `nD`是後跟字`D`的天數。 示例：`P21Y`表示 21 年。 `P1Y2Mo`表示一年零兩個月。 `P1Y2Mo5D`表示一年、兩個月和五天。  `P1Y2M5DT8H5M620S`表示一年、兩個月、五天、八小時、五分鐘和二十秒。  |
|phoneNumber|表示電話號碼。 |
|int| 表示介於 -2，147，483，648 和 2，147，483，647 之間的數位|
|long| 表示介於 -9，223，372，036，854，775，808 到 9，223，372，036，854，775，807 之間 |
|字串| 以一連串的 UTF-16 字碼單位表示文字。|
|stringCollection|表示 `string` 的集合。|
|使用者身份| 表示使用者標識。|
|使用者身份集合|表示 `userIdentity` 的集合。|

### <a name="defaultpartnerclaimtypes"></a>DefaultPartnerClaimTypes

**DefaultPartnerClaimTypes** 可能包含下列元素：

| 元素 | 發生次數 | 描述 |
| ------- | ----------- | ----------- |
| 通訊協定 | 1:n | 含有其預設夥伴宣告類型名稱的通訊協定清單。 |

**Protocol** 元素包含下列屬性：

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| 名稱 | 是 | Azure AD B2C 所支援的有效通訊協定名稱。 可能的值是：OAuth1、OAuth2、SAML2、OpenIdConnect。 |
| PartnerClaimType | 是 | 要使用的宣告類型名稱。 |

在下列範例中，當識別體驗架構與 SAML2 識別提供者或信賴憑證者應用程式進行互動時，會將 **surname** 宣告對應至 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`，與 OpenIdConnect 和 OAuth2 互動時，則會將宣告對應至 `family_name`。

```XML
<ClaimType Id="surname">
  <DisplayName>Surname</DisplayName>
  <DataType>string</DataType>
  <DefaultPartnerClaimTypes>
    <Protocol Name="OAuth2" PartnerClaimType="family_name" />
    <Protocol Name="OpenIdConnect" PartnerClaimType="family_name" />
    <Protocol Name="SAML2" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" />
  </DefaultPartnerClaimTypes>
</ClaimType>
```

因此，Azure AD B2C 所發出的 JWT 權杖會略過 `family_name`，而不會略過 ClaimType 名稱 **surname**。

```JSON
{
  "sub": "6fbbd70d-262b-4b50-804c-257ae1706ef2",
  "auth_time": 1535013501,
  "given_name": "David",
  "family_name": "Williams",
  "name": "David Williams",
}
```

### <a name="mask"></a>Mask

**Mask** 元素包含下列屬性：

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| `Type` | 是 | 宣告遮罩的類型。 可能的值：`Simple` 或 `Regex`。 `Simple` 值表示會將簡單的文字遮罩套用到字串宣告的前置部分。 `Regex` 值表示會將規則運算式套用到整個字串宣告。  如果指定 `Regex` 值，也必須透過要使用的規則運算式來定義選擇性屬性。 |
| `Regex` | 否 | 如果**`Type`** 設置為`Regex`，請指定要使用的正則運算式。

下列範例會使用 `Simple` 遮罩來設定 **PhoneNumber** 宣告：

```XML
<ClaimType Id="PhoneNumber">
  <DisplayName>Phone Number</DisplayName>
  <DataType>string</DataType>
  <Mask Type="Simple">XXX-XXX-</Mask>
  <UserHelpText>Your telephone number.</UserHelpText>
</ClaimType>
```

識別體驗架構會呈現電話號碼，同時隱藏前六個數字：

![瀏覽器中顯示的電話號碼聲明，前六位數位被 Xs 遮罩](./media/claimsschema/mask.png)

下列範例會使用 `Regex` 遮罩來設定 **AlternateEmail** 宣告：

```XML
<ClaimType Id="AlternateEmail">
  <DisplayName>Please verify the secondary email linked to your account</DisplayName>
  <DataType>string</DataType>
  <Mask Type="Regex" Regex="(?&lt;=.).(?=.*@)">*</Mask>
  <UserInputType>Readonly</UserInputType>
</ClaimType>
```

識別體驗架構只會呈現電子郵件地址和電子郵件網域名稱的第一個字母：

![瀏覽器中顯示的電子郵件聲明，字元由星號遮蓋](./media/claimsschema/mask-regex.png)


### <a name="restriction"></a>限制

**Restriction** 元素可以包含下列屬性：

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| MergeBehavior | 否 | 此方法可用來合併列舉值與具備相同識別碼之父代原則中的 ClaimType。 當您覆寫基本原則中指定的宣告時，請使用這個屬性。 可能的值：`Append`、`Prepend` 或 `ReplaceAll`。 `Append` 值是資料集合，應該附加至父代原則中指定的集合結尾。 `Prepend` 值是資料集合，應該新增到父代原則中指定的集合之前。 `ReplaceAll` 值是父代原則中應忽略的指定資料集合。 |

**Restriction** 元素包含下列元素：

| 元素 | 發生次數 | 描述 |
| ------- | ----------- | ----------- |
| 列舉型別 | 1:n | 使用者介面中使用者可用來針對宣告進行選取的可用選項，例如下拉式清單中的值。 |
| 模式 | 1:1 | 要使用的規則運算式。 |

#### <a name="enumeration"></a>列舉型別

枚**舉**元素定義可供使用者為使用者介面中的聲明（如`CheckboxMultiSelect`中`DropdownSingleSelect`的值或`RadioSingleSelect`中的值）選擇的可用選項。 或者，您可以使用[當地語系化集合](localization.md#localizedcollections)元素定義和當地語系化可用選項。 要從聲明**枚舉**集合中查找項，請使用[GetMappedValue 從當地語系化集合](string-transformations.md#getmappedvaluefromlocalizedcollection)聲明轉換。

**Enumeration** 元素包含下列屬性：

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| Text | 是 | 針對此選項，要在使用者介面中顯示給使用者的顯示字串。 |
|值 | 是 | 與選取此選項相關聯的宣告值。 |
| SelectByDefault | 否 | 指出預設是否應該在 UI 中選取此選項。 可能的值：True 或 False。 |

下列範例會設定**城市**下拉式清單宣告，並將預設值設定為 `New York`：

```XML
<ClaimType Id="city">
  <DisplayName>city where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="New York" Value="new-york" SelectByDefault="true" />
  </Restriction>
</ClaimType>
```

預設值設為 New York 的城市下拉式清單：

![在瀏覽器中呈現並顯示預設值的下拉控制項](./media/claimsschema/dropdownsingleselect.png)

### <a name="pattern"></a>模式

**Pattern** 元素可以包含下列屬性：

| 屬性 | 必要 | 描述 |
| --------- | -------- | ----------- |
| RegularExpression | 是 | 此類型的宣告必須符合才能生效的規則運算式。 |
| HelpText | 否 | 如果正則運算式檢查失敗，則給使用者的錯誤訊息。 |

下列範例會設定 **email** 宣告，並提供規則運算式輸入驗證和說明文字：

```XML
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <DefaultPartnerClaimTypes>
    <Protocol Name="OpenIdConnect" PartnerClaimType="email" />
  </DefaultPartnerClaimTypes>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
  <Restriction>
    <Pattern RegularExpression="^[a-zA-Z0-9.!#$%&amp;'^_`{}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$" HelpText="Please enter a valid email address." />
    </Restriction>
 </ClaimType>
```

識別體驗架構會使用電子郵件格式輸入驗證來呈現電子郵件地址宣告：

![顯示由正則運算式限制觸發的錯誤訊息的文字方塊](./media/claimsschema/pattern.png)

### <a name="userinputtype"></a>UserInputType

Azure AD B2C 支援各種不同的使用者輸入類型 (例如文字方塊、密碼與下拉式清單)，可在手動輸入宣告類型的宣告資料時使用。 當您使用[自斷言的技術設定檔](self-asserted-technical-profile.md)和[顯示控制項](display-controls.md)從使用者收集資訊時，必須指定**UserInputType。**

**使用者輸入類型**元素可用使用者輸入類型：

| UserInputType | 支援的聲明類型 | 描述 |
| --------- | -------- | ----------- |
|CheckboxMultiSelect| `string` |多選下拉清單。 聲明值以所選值的逗號分隔符號字串表示。 |
|DateTimeDropdown | `date`, `dateTime` |下拉清單以選擇天、月和年。 |
|DropdownSingleSelect |`string` |單個選擇下拉清單。 聲明值是所選值。|
|EmailBox | `string` |電子郵件輸入欄位。 |
|Paragraph | `boolean`, `date`, `dateTime`, `duration`, `int`, `long`, `string`|僅在段落標記中顯示文本的欄位。 |
|密碼 | `string` |密碼文字方塊。|
|RadioSingleSelect |`string` | 選項按鈕的集合。 聲明值是所選值。|
|Readonly | `boolean`, `date`, `dateTime`, `duration`, `int`, `long`, `string`| 唯讀文字方塊。 |
|TextBox |`boolean`, `int`, `string` |單行文字方塊。 |


#### <a name="textbox"></a>TextBox

**TextBox** 使用者輸入類型會用來提供單行文字方塊。

![顯示聲明類型中指定屬性的文字方塊](./media/claimsschema/textbox.png)

```XML
<ClaimType Id="displayName">
  <DisplayName>Display Name</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Your display name.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

#### <a name="emailbox"></a>EmailBox

**EmailBox** 使用者輸入類型會用來提供基本的電子郵件輸入欄位。

![顯示聲明類型中指定屬性的電子郵件框](./media/claimsschema/emailbox.png)

```XML
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>EmailBox</UserInputType>
  <Restriction>
    <Pattern RegularExpression="^[a-zA-Z0-9!#$%&amp;'+^_`{}~-]+(?:\.[a-zA-Z0-9!#$%&amp;'+^_`{}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?$" HelpText="Please enter a valid email address." />
  </Restriction>
</ClaimType>
```

#### <a name="password"></a>密碼

**Password** 使用者輸入類型會用來記錄使用者所輸入的密碼。

![搭配使用宣告類型與 Password](./media/claimsschema/password.png)

```XML
<ClaimType Id="password">
  <DisplayName>Password</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Enter password</UserHelpText>
  <UserInputType>Password</UserInputType>
</ClaimType>
```

#### <a name="datetimedropdown"></a>DateTimeDropdown

**DateTimeDropdown** 使用者輸入類型會用來提供一組下拉式清單以選取日、月和年。 您可以使用 Predicates 和 PredicateValidations 元素來控制最小和最大日期值。 如需詳細資訊，請參閱 [Predicates 與 PredicateValidations](predicates.md) 的**設定日期範圍**一節。

![搭配使用宣告類型與 DateTimeDropdown](./media/claimsschema/datetimedropdown.png)

```XML
<ClaimType Id="dateOfBirth">
  <DisplayName>Date Of Birth</DisplayName>
  <DataType>date</DataType>
  <UserHelpText>The date on which you were born.</UserHelpText>
  <UserInputType>DateTimeDropdown</UserInputType>
</ClaimType>
```

#### <a name="radiosingleselect"></a>RadioSingleSelect

**RadioSingleSelect** 使用者輸入類型會用來提供可讓使用者選取一個選項的選項按鈕集合。

![搭配使用宣告類型和 RadioSingleSelect](./media/claimsschema/radiosingleselect.png)

```XML
<ClaimType Id="color">
  <DisplayName>Preferred color</DisplayName>
  <DataType>string</DataType>
  <UserInputType>RadioSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Blue" Value="Blue" SelectByDefault="false" />
    <Enumeration Text="Green " Value="Green" SelectByDefault="false" />
    <Enumeration Text="Orange" Value="Orange" SelectByDefault="true" />
  </Restriction>
</ClaimType>
```

#### <a name="dropdownsingleselect"></a>DropdownSingleSelect

**DropdownSingleSelect** 使用者輸入類型會用來提供可讓使用者選取一個選項的下拉式清單方塊。

![搭配使用宣告類型和 DropdownSingleSelect](./media/claimsschema/dropdownsingleselect.png)

```XML
<ClaimType Id="city">
  <DisplayName>City where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="New York" Value="new-york" SelectByDefault="true" />
  </Restriction>
</ClaimType>
```

#### <a name="checkboxmultiselect"></a>CheckboxMultiSelect

**CheckboxMultiSelect** 使用者輸入類型會用來提供可讓使用者選取多個選項的核取方塊集合。

![搭配使用宣告類型和 CheckboxMultiSelect](./media/claimsschema/checkboxmultiselect.png)

```XML
<ClaimType Id="languages">
  <DisplayName>Languages you speak</DisplayName>
  <DataType>string</DataType>
  <UserInputType>CheckboxMultiSelect</UserInputType>
  <Restriction>
    <Enumeration Text="English" Value="English" SelectByDefault="true" />
    <Enumeration Text="France " Value="France" SelectByDefault="false" />
    <Enumeration Text="Spanish" Value="Spanish" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```

#### <a name="readonly"></a>Readonly

**Readonly** 使用者輸入類型會用來提供要顯示宣告和值的唯讀欄位。

![搭配使用宣告類型與 Readonly](./media/claimsschema/readonly.png)

```XML
<ClaimType Id="membershipNumber">
  <DisplayName>Membership number</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Your membership number (read only)</UserHelpText>
  <UserInputType>Readonly</UserInputType>
</ClaimType>
```


#### <a name="paragraph"></a>Paragraph

**Paragraph** 使用者輸入類型會用來提供只能在段落標記中顯示文字的欄位。  例如，&lt;p&gt;text&lt;/p&gt;。 **段落**使用者輸入類型的`OutputClaim`自斷言技術設定檔，必須設置`Required`屬性`false`（預設值）。

![搭配使用宣告類型與 Paragraph](./media/claimsschema/paragraph.png)

```XML
<ClaimType Id="responseMsg">
  <DisplayName>Error message: </DisplayName>
  <DataType>string</DataType>
  <AdminHelpText>A claim responsible for holding response messages to send to the relying party</AdminHelpText>
  <UserHelpText>A claim responsible for holding response messages to send to the relying party</UserHelpText>
  <UserInputType>Paragraph</UserInputType>
  <Restriction>
    <Enumeration Text="B2C_V1_90001" Value="You cannot sign in because you are a minor" />
    <Enumeration Text="B2C_V1_90002" Value="This action can only be performed by gold members" />
    <Enumeration Text="B2C_V1_90003" Value="You have not been enabled for this operation" />
  </Restriction>
</ClaimType>
```
