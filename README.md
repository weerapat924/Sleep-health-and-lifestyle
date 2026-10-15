วิเคราะห์เบื้องต้นจาก data/processed/sleep_clean.csv
ผลลัพธ์: ตารางสรุปใน outputs/ และกราฟใน outputs/figures/
 
วิธีรัน: python code/02_analysis.py
"""
from pathlib import Path
import pandas as pd
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import seaborn as sns
 
ROOT = Path(__file__).resolve().parent.parent
DATA = ROOT / "data" / "processed" / "sleep_clean.csv"
OUT = ROOT / "outputs"
FIG = OUT / "figures"
FIG.mkdir(parents=True, exist_ok=True)
sns.set_theme(style="whitegrid")
 
df = pd.read_csv(DATA)
 
# ---------- 1) สถิติพื้นฐาน ----------
num_cols = ["age", "sleep_hours", "sleep_quality", "activity_minutes",
            "stress_level", "heart_rate", "daily_steps",
            "systolic_bp", "diastolic_bp"]
df[num_cols].describe().T.round(2).to_csv(OUT / "summary_stats.csv")
 
# ---------- 2) เปรียบเทียบตามกลุ่ม ----------
by_disorder = (df.groupby("sleep_disorder")
                 .agg(n=("person_id", "count"),
                      sleep_hours=("sleep_hours", "mean"),
                      sleep_quality=("sleep_quality", "mean"),
                      stress_level=("stress_level", "mean"),
                      daily_steps=("daily_steps", "mean"))
                 .round(2))
by_occ = (df.groupby("occupation")
            .agg(n=("person_id", "count"),
                 sleep_hours=("sleep_hours", "mean"),
                 sleep_quality=("sleep_quality", "mean"),
                 stress_level=("stress_level", "mean"))
            .round(2).sort_values("sleep_quality"))
by_disorder.to_csv(OUT / "by_sleep_disorder.csv")
by_occ.to_csv(OUT / "by_occupation.csv")
print(by_disorder, "\n")
print(by_occ, "\n")
 
# ---------- 3) สหสัมพันธ์ ----------
corr = df[num_cols].corr().round(2)
print("สหสัมพันธ์กับคุณภาพการนอน:")
print(corr["sleep_quality"].drop("sleep_quality").sort_values(), "\n")
 
# ---------- 4) กราฟ ----------
fig, ax = plt.subplots(figsize=(8, 5))
sns.boxplot(data=df, x="sleep_disorder", y="sleep_hours",
            order=["No Disorder", "Insomnia", "Sleep Apnea"], ax=ax)
ax.set(title="Sleep duration by sleep disorder", xlabel="", ylabel="Hours")
fig.tight_layout(); fig.savefig(FIG / "sleep_hours_by_disorder.png", dpi=150)
 
fig, ax = plt.subplots(figsize=(8, 5))
sns.regplot(data=df, x="stress_level", y="sleep_quality",
            x_jitter=0.15, y_jitter=0.15, scatter_kws={"alpha": 0.4}, ax=ax)
ax.set(title="Stress level vs. sleep quality")
fig.tight_layout(); fig.savefig(FIG / "stress_vs_quality.png", dpi=150)
 
fig, ax = plt.subplots(figsize=(9, 7))
sns.heatmap(corr, annot=True, cmap="coolwarm", center=0, fmt=".2f", ax=ax)
ax.set(title="Correlation matrix")
fig.tight_layout(); fig.savefig(FIG / "correlation_heatmap.png", dpi=150)
 
fig, ax = plt.subplots(figsize=(8, 5))
by_occ["sleep_quality"].plot.barh(ax=ax)
ax.set(title="Average sleep quality by occupation", xlabel="Score (1-10)", ylabel="")
fig.tight_layout(); fig.savefig(FIG / "quality_by_occupation.png", dpi=150)
 
print("บันทึกผลลัพธ์ใน outputs/ แล้ว")
 # Sleep-health-and-lifestyle
