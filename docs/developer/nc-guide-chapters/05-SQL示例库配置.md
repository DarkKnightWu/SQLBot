## 5. SQL 示例库配置

### 5.1 凭证查询示例

```yaml
问题: 查询本月所有已记账凭证
SQL: |
  SELECT 
      v.no AS 凭证号,
      v.prepareddate AS 制单日期,
      d.subjcode AS 科目编码,
      d.subjname AS 科目名称,
      d.explanation AS 摘要,
      COALESCE(d.localdebitamount, 0) AS 借方金额,
      COALESCE(d.localcreditamount, 0) AS 贷方金额
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  WHERE v.dr = 0 
    AND d.dr = 0
    AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
  ORDER BY v.no, d.pk_detail
数据源: NC财务系统

问题: 查询指定科目的凭证明细
SQL: |
  SELECT 
      v.prepareddate AS 制单日期,
      v.no AS 凭证号,
      d.explanation AS 摘要,
      COALESCE(d.localdebitamount, 0) AS 借方金额,
      COALESCE(d.localcreditamount, 0) AS 贷方金额,
      a.subjcode AS 科目编码,
      a.subjname AS 科目名称
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 
    AND d.dr = 0
    AND v.posted = 'Y'
    AND a.subjcode LIKE '1001%'
  ORDER BY v.prepareddate, v.no
数据源: NC财务系统

问题: 查询本月金额大于1万元的凭证
SQL: |
  SELECT 
      v.year AS 年度,
      v.period AS 期间,
      v.no AS 凭证号,
      d.explanation AS 摘要,
      d.localdebitamount AS 借方金额
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_corp c ON v.pk_corp = c.pk_corp
  WHERE c.unitname LIKE '%{CORP_NAME}%'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
    AND d.localdebitamount > 10000
    AND v.dr = 0
  ORDER BY d.localdebitamount DESC
数据源: NC财务系统

问题: 查询包含特定关键词的凭证记录
SQL: |
  SELECT 
      v.prepareddate AS 制单日期,
      v.no AS 凭证号,
      d.explanation AS 摘要,
      d.localdebitamount AS 金额,
      a.subjname AS 科目
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.pk_corp = (SELECT pk_corp FROM bd_corp WHERE unitname LIKE '%{CORP_NAME}%')
    AND d.explanation LIKE '%{KEYWORD}%'
    AND v.year = '{YEAR}'
    AND v.dr = 0
数据源: NC财务系统

问题: 查询某制单人经手的所有凭证
SQL: |
  SELECT 
      u.user_name AS 制单人,
      COUNT(v.pk_voucher) AS 凭证数量,
      SUM(v.totaldebit) AS 总借方金额
  FROM gl_voucher v
  JOIN sm_user u ON v.preparer = u.cuserid
  WHERE u.user_name LIKE '%{USER_NAME}%'
    AND v.year = '{YEAR}'
    AND v.dr = 0
  GROUP BY u.user_name
数据源: NC财务系统

问题: 统计本月凭证数量和金额
SQL: |
  SELECT 
      COUNT(DISTINCT v.pk_voucher) AS 凭证数量,
      SUM(COALESCE(d.localdebitamount, 0)) AS 借方合计,
      SUM(COALESCE(d.localcreditamount, 0)) AS 贷方合计
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  WHERE v.dr = 0 
    AND d.dr = 0
    AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
数据源: NC财务系统
```

### 5.2 余额查询示例

```yaml
问题: 查询某公司某年某月的银行存款余额
SQL: |
  SELECT 
      c.unitname AS 公司名称,
      a.subjname AS 科目名称,
      SUM(d.localdebitamount) - SUM(d.localcreditamount) AS 期末余额
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_corp c ON v.pk_corp = c.pk_corp
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE c.unitname LIKE '%{CORP_NAME}%'
    AND v.year = '{YEAR}'
    AND v.period <= '{PERIOD}'
    AND a.subjcode LIKE '1002%'
    AND v.dr = 0
    AND v.posted = 'Y'
  GROUP BY c.unitname, a.subjname
数据源: NC财务系统

问题: 查询某科目期末余额
SQL: |
  SELECT 
      a.subjcode AS 科目编码,
      a.subjname AS 科目名称,
      b.year AS 年度,
      b.period AS 期间,
      b.m_qc AS 期初余额,
      b.m_debit AS 本期借方,
      b.m_credit AS 本期贷方,
      b.m_qm AS 期末余额
  FROM gl_balance b
  INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
  WHERE b.year = '{YEAR}'
    AND b.period = '{PERIOD}'
    AND a.subjcode LIKE '1002%'
    AND a.dr = 0
数据源: NC财务系统

问题: 查询现金及银行存款类科目的余额
SQL: |
  SELECT 
      a.subjname AS 科目名称,
      SUM(b.m_qm) AS 期末余额
  FROM gl_balance b
  JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
  WHERE (a.subjcode LIKE '1001%' OR a.subjcode LIKE '1002%')
    AND b.year = '{YEAR}'
    AND b.period = '{PERIOD}'
  GROUP BY a.subjname
数据源: NC财务系统

问题: 查询各一级科目余额汇总
SQL: |
  SELECT 
      SUBSTR(a.subjcode, 1, 4) AS 科目大类,
      a.subjname AS 科目名称,
      SUM(b.m_qm) AS 期末余额合计
  FROM gl_balance b
  INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
  WHERE b.year = '{YEAR}'
    AND b.period = '{PERIOD}'
    AND a.subjlevel = 1
    AND a.dr = 0
  GROUP BY SUBSTR(a.subjcode, 1, 4), a.subjname
  ORDER BY 科目大类
数据源: NC财务系统

问题: 查询资产负债率
SQL: |
  WITH asset_total AS (
      SELECT SUM(b.m_qm) AS total_asset
      FROM gl_balance b
      INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
      WHERE b.year = '{YEAR}'
        AND b.period = '{PERIOD}'
        AND a.subjcode LIKE '1%'
        AND a.endflag = 'Y'
        AND a.dr = 0
  ),
  liability_total AS (
      SELECT SUM(b.m_qm) AS total_liability
      FROM gl_balance b
      INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
      WHERE b.year = '{YEAR}'
        AND b.period = '{PERIOD}'
        AND a.subjcode LIKE '2%'
        AND a.endflag = 'Y'
        AND a.dr = 0
  )
  SELECT 
      a.total_asset AS 资产总额,
      l.total_liability AS 负债总额,
      ROUND(l.total_liability / NULLIF(a.total_asset, 0) * 100, 2) AS 资产负债率
  FROM asset_total a, liability_total l
数据源: NC财务系统
```

### 5.3 费用统计示例

```yaml
问题: 统计某公司年度总管理费用
SQL: |
  SELECT 
      SUM(d.localdebitamount) AS 年度管理费用总额
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  JOIN bd_corp c ON v.pk_corp = c.pk_corp
  WHERE c.unitname LIKE '%{CORP_NAME}%'
    AND v.year = '{YEAR}'
    AND a.subjcode LIKE '6602%'
    AND v.dr = 0
数据源: NC财务系统

问题: 按部门统计某年度的差旅费支出排行
SQL: |
  SELECT 
      dept.deptname AS 部门名称,
      SUM(d.localdebitamount) AS 差旅费总额
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_deptdoc dept ON d.pk_deptid = dept.pk_deptdoc
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.pk_corp = (SELECT pk_corp FROM bd_corp WHERE unitname LIKE '%{CORP_NAME}%')
    AND v.year = '{YEAR}'
    AND a.subjname LIKE '%差旅费%'
    AND v.dr = 0
  GROUP BY dept.deptname
  ORDER BY SUM(d.localdebitamount) DESC
数据源: NC财务系统

问题: 查询某个部门在某个月发生的费用总额
SQL: |
  SELECT 
      d.pk_deptdoc,
      dept.deptname AS 部门名称,
      SUM(d.m_debit) AS 发生额
  FROM gl_detail d
  JOIN bd_deptdoc dept ON d.pk_deptdoc = dept.pk_deptdoc
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE a.subjcode LIKE '6602%'
    AND d.year = '{YEAR}'
    AND d.period = '{PERIOD}'
  GROUP BY d.pk_deptdoc, dept.deptname
数据源: NC财务系统

问题: 按部门统计管理费用
SQL: |
  SELECT 
      dept.deptname AS 部门名称,
      SUM(COALESCE(d.localdebitamount, 0) - COALESCE(d.localcreditamount, 0)) AS 管理费用
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_deptdoc dept ON d.pk_deptid = dept.pk_deptdoc
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
    AND a.subjcode LIKE '6602%'
  GROUP BY dept.deptname
  ORDER BY 管理费用 DESC
数据源: NC财务系统
```

### 5.4 应收应付示例

```yaml
问题: 查询应收账款账龄分析
SQL: |
  SELECT 
      c.custname AS 客户名称,
      COUNT(*) AS 单据数量,
      SUM(CASE WHEN SYSDATE - ar.billdate <= 30 THEN ar.money_bal ELSE 0 END) AS "0-30天",
      SUM(CASE WHEN SYSDATE - ar.billdate BETWEEN 31 AND 60 THEN ar.money_bal ELSE 0 END) AS "31-60天",
      SUM(CASE WHEN SYSDATE - ar.billdate BETWEEN 61 AND 90 THEN ar.money_bal ELSE 0 END) AS "61-90天",
      SUM(CASE WHEN SYSDATE - ar.billdate > 90 THEN ar.money_bal ELSE 0 END) AS "90天以上",
      SUM(ar.money_bal) AS 合计金额
  FROM ar_recbill ar
  INNER JOIN bd_customer c ON ar.pk_customer = c.pk_customer
  WHERE ar.dr = 0
    AND ar.money_bal > 0
  GROUP BY c.custname
  ORDER BY 合计金额 DESC
数据源: NC财务系统

问题: 查询某客户的应收账款欠款明细
SQL: |
  SELECT 
      c.custname AS 客户名称,
      ar.billno AS 单据号,
      ar.money_bal AS 未核销金额,
      ar.billdate AS 单据日期
  FROM ar_recbill ar
  JOIN bd_customer c ON ar.pk_customer = c.pk_customer
  WHERE c.custname LIKE '%{CUST_NAME}%'
    AND ar.money_bal > 0
数据源: NC财务系统

问题: 查询某个供应商的应付账款未核销余额
SQL: |
  SELECT 
      s.supname AS 供应商名称,
      SUM(ap.money_bal) AS 未核销金额
  FROM ap_paybill ap
  JOIN bd_supplier s ON ap.pk_supplier = s.pk_supplier
  WHERE s.supname LIKE '%{SUP_NAME}%'
    AND ap.billstatus >= 2
  GROUP BY s.supname
数据源: NC财务系统

问题: 查询本月付款明细
SQL: |
  SELECT 
      p.billno AS 单据编号,
      p.paydate AS 付款日期,
      s.supname AS 供应商名称,
      p.payamount AS 付款金额,
      cur.currname AS 币种
  FROM ap_paybill p
  INNER JOIN bd_supplier s ON p.pk_supplier = s.pk_supplier
  LEFT JOIN bd_currtype cur ON p.pk_currtype = cur.pk_currtype
  WHERE p.dr = 0
    AND p.billstatus >= 2
    AND TO_CHAR(p.paydate, 'YYYY') = '{YEAR}'
    AND TO_CHAR(p.paydate, 'MM') = '{PERIOD}'
  ORDER BY p.paydate DESC
数据源: NC财务系统

问题: 按客户统计应收账款
SQL: |
  SELECT 
      c.custcode AS 客户编码,
      c.custname AS 客户名称,
      SUM(b.m_qm) AS 应收余额
  FROM gl_accass b
  INNER JOIN bd_cubasdoc c ON b.pk_assid = c.pk_cubasdoc
  INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
  WHERE b.year = '{YEAR}'
    AND b.period = '{PERIOD}'
    AND a.subjcode LIKE '1122%'
    AND b.dr = 0
  GROUP BY c.custcode, c.custname
  HAVING SUM(b.m_qm) <> 0
  ORDER BY 应收余额 DESC
数据源: NC财务系统

问题: 按供应商统计应付账款
SQL: |
  SELECT 
      s.supcode AS 供应商编码,
      s.supname AS 供应商名称,
      SUM(b.m_qm) AS 应付余额
  FROM gl_accass b
  INNER JOIN bd_supbasdoc s ON b.pk_assid = s.pk_supbasdoc
  INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
  WHERE b.year = '{YEAR}'
    AND b.period = '{PERIOD}'
    AND a.subjcode LIKE '2202%'
    AND b.dr = 0
  GROUP BY s.supcode, s.supname
  HAVING SUM(b.m_qm) <> 0
  ORDER BY 应付余额 DESC
数据源: NC财务系统
```

### 5.5 员工与个人往来示例

```yaml
问题: 查询某员工的借款余额
SQL: |
  SELECT 
      p.psnname AS 员工姓名,
      SUM(d.localdebitamount) - SUM(d.localcreditamount) AS 当前借款余额
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_psndoc p ON d.pk_psndoc = p.pk_psndoc
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE p.psnname LIKE '%{PERSON_NAME}%'
    AND a.subjcode LIKE '1221%'
    AND v.dr = 0
    AND v.posted = 'Y'
  GROUP BY p.psnname
数据源: NC财务系统
```

### 5.6 项目核算示例

```yaml
问题: 按项目统计全年的收入与成本
SQL: |
  SELECT 
      proj.project_name AS 项目名称,
      SUM(CASE WHEN a.subjcode LIKE '6001%' THEN d.localcreditamount ELSE 0 END) AS 项目收入,
      SUM(CASE WHEN a.subjcode LIKE '6401%' THEN d.localdebitamount ELSE 0 END) AS 项目成本
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_project proj ON d.pk_project = proj.pk_project
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.year = '{YEAR}'
    AND v.dr = 0
  GROUP BY proj.project_name
数据源: NC财务系统

问题: 项目成本归集分析
SQL: |
  SELECT 
      p.project_code AS 项目编码,
      p.project_name AS 项目名称,
      SUM(CASE WHEN a.subjcode LIKE '6401%' THEN COALESCE(d.localdebitamount, 0) ELSE 0 END) AS 直接材料,
      SUM(CASE WHEN a.subjcode LIKE '6402%' THEN COALESCE(d.localdebitamount, 0) ELSE 0 END) AS 直接人工,
      SUM(CASE WHEN a.subjcode LIKE '6403%' THEN COALESCE(d.localdebitamount, 0) ELSE 0 END) AS 制造费用,
      SUM(COALESCE(d.localdebitamount, 0)) AS 成本合计
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_project p ON d.pk_project = p.pk_project
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND a.subjcode LIKE '64%'
  GROUP BY p.project_code, p.project_name
  ORDER BY 成本合计 DESC
数据源: NC财务系统
```

### 5.7 固定资产示例

```yaml
问题: 查询固定资产明细及净值
SQL: |
  SELECT 
      fa.assetcode AS 资产编码,
      fa.assetname AS 资产名称,
      fa.originalvalue AS 原值,
      fa.accudepre AS 累计折旧,
      fa.originalvalue - fa.accudepre AS 净值,
      fa.startdate AS 入账日期,
      fa.deptname AS 使用部门
  FROM fa_card fa
  WHERE fa.dr = 0
    AND fa.cardstatus = 1
  ORDER BY fa.originalvalue DESC
数据源: NC财务系统

问题: 统计各部门固定资产总值
SQL: |
  SELECT 
      fa.deptname AS 使用部门,
      COUNT(*) AS 资产数量,
      SUM(fa.originalvalue) AS 资产原值合计,
      SUM(fa.accudepre) AS 累计折旧合计,
      SUM(fa.originalvalue - fa.accudepre) AS 净值合计
  FROM fa_card fa
  WHERE fa.dr = 0
    AND fa.cardstatus = 1
  GROUP BY fa.deptname
  ORDER BY 净值合计 DESC
数据源: NC财务系统
```

### 5.8 财务报表类示例

```yaml
问题: 查询资产负债表主要项目
SQL: |
  WITH asset_items AS (
      SELECT 
          CASE 
              WHEN a.subjcode LIKE '1001%' THEN '货币资金'
              WHEN a.subjcode LIKE '1002%' THEN '货币资金'
              WHEN a.subjcode LIKE '1122%' THEN '应收账款'
              WHEN a.subjcode LIKE '1123%' THEN '预付账款'
              WHEN a.subjcode LIKE '1401%' THEN '存货'
              WHEN a.subjcode LIKE '1601%' THEN '固定资产'
              WHEN a.subjcode LIKE '1701%' THEN '无形资产'
              ELSE '其他资产'
          END AS 项目名称,
          SUM(b.m_qm) AS 期末余额
      FROM gl_balance b
      INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
      WHERE b.year = '{YEAR}'
        AND b.period = '{PERIOD}'
        AND a.subjcode LIKE '1%'
        AND a.endflag = 'Y'
        AND a.dr = 0
      GROUP BY CASE 
          WHEN a.subjcode LIKE '1001%' THEN '货币资金'
          WHEN a.subjcode LIKE '1002%' THEN '货币资金'
          WHEN a.subjcode LIKE '1122%' THEN '应收账款'
          WHEN a.subjcode LIKE '1123%' THEN '预付账款'
          WHEN a.subjcode LIKE '1401%' THEN '存货'
          WHEN a.subjcode LIKE '1601%' THEN '固定资产'
          WHEN a.subjcode LIKE '1701%' THEN '无形资产'
          ELSE '其他资产'
      END
  )
  SELECT 项目名称, 期末余额
  FROM asset_items
  ORDER BY 期末余额 DESC
数据源: NC财务系统

问题: 生成本月利润表
SQL: |
  SELECT 
      '一、营业收入' AS 项目,
      SUM(CASE WHEN a.subjcode LIKE '5001%' 
          THEN COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0) 
          ELSE 0 END) AS 本期金额
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
  
  UNION ALL
  
  SELECT 
      '二、营业成本' AS 项目,
      SUM(CASE WHEN a.subjcode LIKE '6001%' 
          THEN COALESCE(d.localdebitamount, 0) - COALESCE(d.localcreditamount, 0) 
          ELSE 0 END) AS 本期金额
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
  
  UNION ALL
  
  SELECT 
      '三、销售费用' AS 项目,
      SUM(CASE WHEN a.subjcode LIKE '6601%' 
          THEN COALESCE(d.localdebitamount, 0) - COALESCE(d.localcreditamount, 0) 
          ELSE 0 END) AS 本期金额
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
  
  UNION ALL
  
  SELECT 
      '四、管理费用' AS 项目,
      SUM(CASE WHEN a.subjcode LIKE '6602%' 
          THEN COALESCE(d.localdebitamount, 0) - COALESCE(d.localcreditamount, 0) 
          ELSE 0 END) AS 本期金额
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
  
  UNION ALL
  
  SELECT 
      '五、财务费用' AS 项目,
      SUM(CASE WHEN a.subjcode LIKE '6603%' 
          THEN COALESCE(d.localdebitamount, 0) - COALESCE(d.localcreditamount, 0) 
          ELSE 0 END) AS 本期金额
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
数据源: NC财务系统
```

### 5.9 趋势分析示例

```yaml
问题: 查询某科目的月度发生额趋势
SQL: |
  SELECT 
      v.period AS 月份,
      SUM(d.localdebitamount) AS 月度发生额
  FROM gl_detail d
  JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
  JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  JOIN bd_corp c ON v.pk_corp = c.pk_corp
  WHERE c.unitname LIKE '%{CORP_NAME}%'
    AND v.year = '{YEAR}'
    AND a.subjname LIKE '%{SUBJ_NAME}%'
    AND v.dr = 0
  GROUP BY v.period
  ORDER BY v.period
数据源: NC财务系统

问题: 各月收入趋势分析
SQL: |
  SELECT 
      v.period AS 月份,
      SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0)) AS 月收入,
      SUM(SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0))) 
          OVER (ORDER BY v.period) AS 累计收入
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND a.subjcode LIKE '5001%'
  GROUP BY v.period
  ORDER BY v.period
数据源: NC财务系统

问题: 计算本月销售收入的环比增长率
SQL: |
  WITH MonthlySales AS (
      SELECT 
          v.period,
          SUM(d.localcreditamount) AS sales_amt
      FROM gl_detail d
      JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
      JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
      WHERE a.subjcode LIKE '6001%'
        AND v.year = '{YEAR}'
        AND v.dr = 0
      GROUP BY v.period
  )
  SELECT 
      curr.period AS 月份,
      curr.sales_amt AS 本月收入,
      prev.sales_amt AS 上月收入,
      ROUND((curr.sales_amt - prev.sales_amt) / NULLIF(prev.sales_amt, 0) * 100, 2) AS 环比增长率
  FROM MonthlySales curr
  LEFT JOIN MonthlySales prev ON TO_NUMBER(curr.period) = TO_NUMBER(prev.period) + 1
  ORDER BY curr.period
数据源: NC财务系统

问题: 收入同比分析
SQL: |
  WITH current_year AS (
      SELECT SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0)) AS amount
      FROM gl_voucher v
      INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
      INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
      WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
        AND v.year = '{YEAR}'
        AND v.period = '{PERIOD}'
        AND a.subjcode LIKE '5001%'
  ),
  last_year AS (
      SELECT SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0)) AS amount
      FROM gl_voucher v
      INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
      INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
      WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
        AND v.year = TO_CHAR(TO_NUMBER('{YEAR}') - 1)
        AND v.period = '{PERIOD}'
        AND a.subjcode LIKE '5001%'
  )
  SELECT 
      c.amount AS 本月收入,
      l.amount AS 去年同月收入,
      c.amount - COALESCE(l.amount, 0) AS 同比增减额,
      ROUND((c.amount - COALESCE(l.amount, 0)) / NULLIF(l.amount, 0) * 100, 2) AS 同比增长率
  FROM current_year c, last_year l
数据源: NC财务系统
```

### 5.10 TOP N 排行榜查询

```yaml
问题: 查询前十大供应商的应付账款余额
SQL: |
  SELECT * FROM (
      SELECT 
          sup.supname AS 供应商名称,
          SUM(d.localcreditamount) - SUM(d.localdebitamount) AS 应付余额
      FROM gl_detail d
      JOIN gl_voucher v ON d.pk_voucher = v.pk_voucher
      JOIN bd_supbasdoc sup ON d.assid = sup.pk_supbasdoc
      JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
      WHERE a.subjcode LIKE '2202%'
        AND v.pk_corp = (SELECT pk_corp FROM bd_corp WHERE unitname LIKE '%{CORP_NAME}%')
        AND v.dr = 0
      GROUP BY sup.supname
      ORDER BY 应付余额 DESC
  ) WHERE ROWNUM <= 10
数据源: NC财务系统

问题: 统计前十大欠款客户及金额
SQL: |
  SELECT * FROM (
      SELECT 
          c.custname AS 客户名称,
          SUM(ar.money_bal) AS 总欠款
      FROM ar_recbill ar
      JOIN bd_customer c ON ar.pk_customer = c.pk_customer
      WHERE ar.money_bal > 0
      GROUP BY c.custname
      ORDER BY 总欠款 DESC
  ) WHERE ROWNUM <= 10
数据源: NC财务系统
```

### 5.11 基础档案查询示例

```yaml
问题: 查询特定名称的公司/主体信息
SQL: |
  SELECT 
      unitname AS 公司名称,
      unitcode AS 公司编码,
      pk_corp AS 公司主键
  FROM bd_corp
  WHERE unitname LIKE '%{KEYWORD}%'
数据源: NC财务系统

问题: 查询包含特定名称的会计科目编码
SQL: |
  SELECT 
      subjname AS 科目名称,
      subjcode AS 科目编码,
      pk_accsubj AS 科目主键
  FROM bd_accsubj
  WHERE subjname LIKE '%{KEYWORD}%'
    AND pk_corp = '{PK_CORP}'
数据源: NC财务系统

问题: 查询特定部门的名称和负责人
SQL: |
  SELECT 
      deptname AS 部门名称,
      deptcode AS 部门编码,
      principal AS 负责人
  FROM bd_deptdoc
  WHERE deptname LIKE '%{KEYWORD}%'
数据源: NC财务系统

问题: 查询客户或供应商的基础档案信息
SQL: |
  SELECT 
      custname AS 客商名称,
      custcode AS 客商编码
  FROM bd_cubasdoc
  WHERE custname LIKE '%{KEYWORD}%'
数据源: NC财务系统
```

### 5.12 多组织对比示例

```yaml
问题: 各公司收入对比
SQL: |
  SELECT 
      c.unitname AS 公司名称,
      SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0)) AS 收入金额,
      ROUND(SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0)) * 100.0 / 
            SUM(SUM(COALESCE(d.localcreditamount, 0) - COALESCE(d.localdebitamount, 0))) OVER (), 2) AS 占比
  FROM gl_voucher v
  INNER JOIN gl_detail d ON v.pk_voucher = d.pk_voucher
  INNER JOIN bd_accsubj a ON d.pk_accsubj = a.pk_accsubj
  INNER JOIN bd_corp c ON v.pk_corp = c.pk_corp
  WHERE v.dr = 0 AND d.dr = 0 AND v.posted = 'Y'
    AND v.year = '{YEAR}'
    AND v.period = '{PERIOD}'
    AND a.subjcode LIKE '5001%'
  GROUP BY c.unitname
  ORDER BY 收入金额 DESC
数据源: NC财务系统

问题: 集团合并资产负债
SQL: |
  SELECT 
      CASE 
          WHEN a.subjcode LIKE '1%' THEN '资产'
          WHEN a.subjcode LIKE '2%' THEN '负债'
          WHEN a.subjcode LIKE '3%' THEN '所有者权益'
      END AS 类别,
      SUM(b.m_qm) AS 合并金额
  FROM gl_balance b
  INNER JOIN bd_accsubj a ON b.pk_accsubj = a.pk_accsubj
  WHERE b.year = '{YEAR}'
    AND b.period = '{PERIOD}'
    AND a.subjlevel = 1
    AND a.dr = 0
  GROUP BY CASE 
      WHEN a.subjcode LIKE '1%' THEN '资产'
      WHEN a.subjcode LIKE '2%' THEN '负债'
      WHEN a.subjcode LIKE '3%' THEN '所有者权益'
  END
  ORDER BY 类别
数据源: NC财务系统
```
