# 생산 DB X Python 미니 프로젝트 (상-2. 고장 전 센서 신호 확인)

## 주제: "설비고장 나기 전에 온도나 진동에 이상 신호가 있었는지 확인해봐." 

> "설비고장 나기 전에 온도나 진동에 이상 신호가 있었는지 확인해봐." 

    **1. 지시 해석** — 무엇을 보고해야 하는지 한 줄로: 설비 고장 전 시기, 온도, 진동 등의 로그 데이터 시각화
    고장남 - 고장 전, 온도, 진동 로그 추출, 고장 시기 


    **2. 설계** — 필요한 테이블 · 컬럼 · 계산식: 
    - 테이블: 온도- sensor_hourly-temp_c / 진동- sensor_hourly-vibration_mm_s / 설비 정보-equipment_master / 시기-sensor_hourly-measured_at / 고장 설비 및 시기-production_result
    - 계산식: 불량률(defect_qty/prod_qty), 선택 불량 지점을 기준으로 기간을 나눔()

### Trouble Shooting
1. q("select result_id, prod_qty, defect_qty, round(defect_qty / prod_qty) as 불량률" \
" from production_result") -> 정수형 인자끼리 나눴을 때 몫이 없으면 0으로 결과 나옴 -> as float으로 형변환

2. 불량률 높은 지점을 임계치(k= 2, 3)를 기준으로 뽑기 -> k=2기준, 4개 result_id 추출가능
따라서 k=2로 설정.

3. seneor_hourly join equipment_master on equip_code

4. sensor_hourly를 production_result의 line 값에 맞춰서 값 뽑기

5. equip_code 열을 quality_judgment에 삽입하기 위해서 권한 설정 변경
    alter table quality_judgment add column equip_code varchar(10) references equipment_master(equip_code);
    alter table quality_judgment drop constraint quality_judgment_work_date_line_code_key;
    alter table quality_judgment add unique (work_date, equip_code);


5. 지점을 기준으로 설비오류 나누기

