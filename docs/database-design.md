# 数据库设计文档

## 核心实体关系

```
User (用户)
  └── CaseAssignment (案件分配)

LoanContract (贷款合同)
  └── Case (案件)
        ├── CollectionRecord (催收记录)
        └── RepaymentRecord (还款记录)

Customer (借款人)
  └── LoanContract (贷款合同)
```

## 表结构设计

### users（用户表）
| 字段 | 类型 | 说明 |
|------|------|------|
| id | String (uuid) | 主键 |
| name | String | 姓名 |
| email | String | 邮箱（唯一） |
| password | String | 密码（hash） |
| role | Enum | ADMIN |
| createdAt | DateTime | 创建时间 |
| updatedAt | DateTime | 更新时间 |

### customers（借款人表）
| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | 主键 |
| name | String | 姓名 |
| idCard | String | 身份证号 |
| phone | String | 手机号 |
| address | String? | 地址 |
| createdAt | DateTime | 创建时间 |

### loan_contracts（贷款合同表）
| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | 主键 |
| contractNo | String | 合同编号（唯一） |
| customerId | String | 借款人ID |
| loanType | Enum | PERSONAL / CAR |
| principal | Decimal | 贷款本金 |
| interestRate | Decimal | 利率 |
| term | Int | 贷款期限（月） |
| startDate | DateTime | 放款日期 |
| endDate | DateTime | 到期日期 |
| status | Enum | NORMAL / OVERDUE / BAD_DEBT / CLOSED |
| createdAt | DateTime | 创建时间 |

### cases（案件表）
| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | 主键 |
| caseNo | String | 案件编号（唯一） |
| contractId | String | 合同ID |
| overdueDays | Int | 逾期天数 |
| overdueAmount | Decimal | 逾期金额 |
| stage | Enum | M1/M2/M3/M6/M12/LEGAL |
| status | Enum | OPEN / IN_PROGRESS / CLOSED / WRITTEN_OFF |
| assignedTo | String? | 负责人ID |
| createdAt | DateTime | 创建时间 |
| updatedAt | DateTime | 更新时间 |

### collection_records（催收记录表）
| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | 主键 |
| caseId | String | 案件ID |
| operatorId | String | 操作人ID |
| method | Enum | PHONE / SMS / VISIT / LETTER / LEGAL |
| result | Enum | CONTACTED / NO_ANSWER / PROMISED / REFUSED |
| note | String? | 备注 |
| collectedAt | DateTime | 催收时间 |

### repayment_records（还款记录表）
| 字段 | 类型 | 说明 |
|------|------|------|
| id | String | 主键 |
| caseId | String | 案件ID |
| amount | Decimal | 还款金额 |
| method | Enum | BANK / CASH / WECHAT / ALIPAY |
| repaidAt | DateTime | 还款日期 |
| note | String? | 备注 |
| createdBy | String | 录入人ID |
| createdAt | DateTime | 创建时间 |
