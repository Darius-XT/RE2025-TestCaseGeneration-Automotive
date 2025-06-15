## Step one: 需求文档 → 流程图

```
Speed_0_3
If Speed_V is not equal to 1, then Speed_S is set to 3.
Otherwise, if Motor_S > 0, then Speed_S is set to ESP_S; otherwise, Speed_S is set to 5.
```

<img src="C:\Users\XT\AppData\Roaming\Typora\typora-user-images\image-20250615164400246.png" alt="image-20250615164400246" style="zoom:50%;" />

```
State_A_5_1
If the System_State is 'Active', then Torque_F is set to 1.
If the System_State is 'Idle', then Torque_F is set to 0.5.
Otherwise, if the System_State is 'Error', then Torque_F is set to 0.
```

<img src="C:\Users\XT\AppData\Roaming\Typora\typora-user-images\image-20250615155040710.png" alt="image-20250615155040710" style="zoom:50%;" />

```
Torque_N_A_6_1
This module calculates creep-related parameters based on torque and load conditions.
First, it evaluates the Torque_F flag.
If Torque_F is equal to 1, it then assesses the Load_Condition.
If Speed_S < 4, Creep_T is set to the value of Load_Factor.
Otherwise, Creep_S is set to Default_Creep.
```

<img src="C:\Users\XT\AppData\Roaming\Typora\typora-user-images\image-20250615164623464.png" alt="image-20250615164623464" style="zoom:50%;" />

```
Torque_W_A_6_1
This module determines the final Output_Torque based on a series of prioritized conditions.
First, it checks for a critical safety state: if Emergency_Stop_Active is true, Output_Torque is immediately set to 0.
Otherwise, it checks if a Manual_Override_Engaged signal is active. If so, Output_Torque is set to Manual_Torque_Value.
If neither of the above conditions are met, the system evaluates the current Load_Level. If Load_Level exceeds Critical_Load_Limit, Output_Torque is set to Overload_Torque.
If none of the preceding conditions apply, the system then considers creep compensation: if Creep_T is greater than Threshold_Value, Output_Torque is set to Creep_Compensated_Torque.
In all other remaining cases (i.e., if Creep_T is not greater than Threshold_Value and previous conditions were not met), the module completes its execution without further modifying Output_Torque.
```

<img src="C:\Users\XT\Downloads\image-20250615155719018.png" alt="image-20250615155719018" style="zoom:50%;" />

## 依赖关系提取

### 路径分析(生产-消费关系分析)

**Speed_0_3 路径分析**

该流程图共包含 **3 条** 独立路径。

**1. Speed_0_3.1**

- **消费:** `Speed_V [!= 1]`
- **生产:** `Speed_S [= 3]`

**2. Speed_0_3.2**

- **消费:** `Speed_V [== 1]`
- **消费:** `Motor_S [> 0]`
- **消费:** `ESP_S [任意数值]`
- **生产:** `Speed_S [任意数值]` (因为 `Speed_S` 被设置为 `ESP_S` 的值，而 `ESP_S` 是任意数值)

**3. Speed_0_3.3**

- **消费:** `Speed_V [== 1]`
- **消费:** `Motor_S [<= 0]`
- **生产:** `Speed_S [= 5]`



**State_A_5_1 路径分析**

该流程图共包含 **4 条** 独立路径。

**1. State_A_5_1.1**

- **消费:** `System_State ['Active']`
- **生产:** `Torque_F [= 1]`

**2. State_A_5_1.2**

- **消费:** `System_State ['Idle']`
- **生产:** `Torque_F [= 0.5]`

**3. State_A_5_1.3**

- **消费:** `System_State ['Error']`
- **生产:** `Torque_F [= 0]`

**4. State_A_5_1.4**

- **消费:** `System_State [!= 'Active' 且 != 'Idle' 且 != 'Error']`
- **生产:** 无



**Torque_N_A_6_1 路径分析**

该流程图共包含 **3 条** 独立路径。

**1. Torque_N_A_6_1.1**

- **消费:** `Torque_F [!= 1]`
- **生产:** 无

**2. Torque_N_A_6_1.2**

- **消费:** `Torque_F [== 1]`
- **消费:** `Speed_S [< 4]`
- **消费:** `Load_Factor [任意数值]`
- **生产:** `Creep_T [任意数值]` (因为 `Creep_T` 被设置为 `Load_Factor` 的值，而 `Load_Factor` 是任意数值)

**3. Torque_N_A_6_1.3**

- **消费:** `Torque_F [== 1]`
- **消费:** `Speed_S [>= 4]`
- **消费:** `Default_Creep [任意数值]`
- **生产:** `Creep_S [任意数值]` (因为 `Creep_S` 被设置为 `Default_Creep` 的值，而 `Default_Creep` 是任意数值)



**orque_W_A_6_1 路径分析**

该流程图共包含 **5 条** 独立路径。

**1. Torque_W_A_6_1.1**

- **消费:** `Emergency_Stop_Active [true]`
- **生产:** `Output_Torque [= 0]`

**2. Torque_W_A_6_1.2**

- **消费:** `Emergency_Stop_Active [false]`
- **消费:** `Manual_Override_Engaged [true]`
- **消费:** `Manual_Torque_Value [任意数值]`
- **生产:** `Output_Torque [任意数值]` (因为 `Output_Torque` 被设置为 `Manual_Torque_Value` 的值，而 `Manual_Torque_Value` 是任意数值)

**3. Torque_W_A_6_1.3**

- **消费:** `Emergency_Stop_Active [false]`
- **消费:** `Manual_Override_Engaged [false]`
- **消费:** `Load_Level [> Critical_Load_Limit]`
- **消费:** `Critical_Load_Limit [任意数值]`
- **消费:** `Overload_Torque [任意数值]`
- **生产:** `Output_Torque [任意数值]` (因为 `Output_Torque` 被设置为 `Overload_Torque` 的值，而 `Overload_Torque` 是任意数值)

**4. Torque_W_A_6_1.4**

- **消费:** `Emergency_Stop_Active [false]`
- **消费:** `Manual_Override_Engaged [false]`
- **消费:** `Load_Level [<= Critical_Load_Limit]`
- **消费:** `Critical_Load_Limit [任意数值]`
- **消费:** `Creep_T [> Threshold_Value]`
- **消费:** `Threshold_Value [任意数值]`
- **消费:** `Creep_Compensated_Torque [任意数值]`
- **生产:** `Output_Torque [任意数值]` (因为 `Output_Torque` 被设置为 `Creep_Compensated_Torque` 的值，而 `Creep_Compensated_Torque` 是任意数值)

**5. Torque_W_A_6_1.5**

- **消费:** `Emergency_Stop_Active [false]`
- **消费:** `Manual_Override_Engaged [false]`
- **消费:** `Load_Level [<= Critical_Load_Limit]`
- **消费:** `Critical_Load_Limit [任意数值]`
- **消费:** `Creep_T [<= Threshold_Value]`
- **消费:** `Threshold_Value [任意数值]`
- **生产:** 无 (此路径未修改 `Output_Torque`)

### 按变量名得到初步依赖关系

1. `dep :=< Speed_0_3.3, produce, Speed_S = 3, Speed_S < 4, consumedBy, Torque_N_A_6_1.1 >`
2. `dep :=< Speed_0_3.1, produce, Speed_S = (-inf, inf), Speed_S < 4, consumedBy, Torque_N_A_6_1.1 >`
3. `dep :=< Speed_0_3.2, produce, Speed_S = 5, Speed_S < 4, consumedBy, Torque_N_A_6_1.1 >`
4. `dep :=< State_A_5_1.1, produce, Torque_F = 1, Torque_F = 1, consumedBy, Torque_N_A_6_1.1 >`
5. `dep :=< State_A_5_1.2, produce, Torque_F = 0.5, Torque_F == 1, consumedBy, Torque_N_A_6_1.1 >`
6. `dep :=< State_A_5_1.3, produce, Torque_F = 0, Torque_F == 1, consumedBy, Torque_N_A_6_1.1 >`
7. `dep :=< Torque_N_A_6_1.3, produce, Creep_T = (-inf, inf), Creep_T > Threshold_Value, consumedBy, Torque_W_A_6_1.4 >`

### 对值域进行筛选

1. `dep :=< Speed_0_3.3, produce, Speed_S = 3, Speed_S < 4, consumedBy, Torque_N_A_6_1.1 >`
2. `dep :=< Speed_0_3.1, produce, Speed_S = (-inf, inf), Speed_S < 4, consumedBy, Torque_N_A_6_1.1 >`
3. `dep :=< State_A_5_1.1, produce, Torque_F = 1, Torque_F = 1, consumedBy, Torque_N_A_6_1.1 >`
4. `dep :=< Torque_N_A_6_1.3, produce, Creep_T = (-inf, inf), Creep_T > Threshold_Value, consumedBy, Torque_W_A_6_1.4 >`

<img src="C:\Users\XT\AppData\Roaming\Typora\typora-user-images\image-20250615170954890.png" alt="image-20250615170954890" style="zoom:50%;" />

### 得到测试场景

<img src="C:\Users\XT\AppData\Roaming\Typora\typora-user-images\image-20250615171016591.png" alt="image-20250615171016591" style="zoom:50%;" />
