import streamlit as st
import pandas as pd
import plotly.express as px

# ===================== KONFIGURASI HALAMAN =====================
st.set_page_config(
    page_title="Dashboard Survei Hobi",
    page_icon="🎯",
    layout="wide"
)

# ===================== LOAD DATA =====================
@st.cache_data
def load_data():
    df = pd.read_csv("Data_seputar_Hobi.csv")
    # Bersihkan nama kolom (hapus spasi berlebih di awal/akhir)
    df.columns = [c.strip() for c in df.columns]
    # Buang kolom yang seluruhnya kosong
    df = df.dropna(axis=1, how="all")
    return df

df = load_data()

# Ganti nama kolom agar lebih singkat & mudah dipakai di kode
rename_map = {
    "JENIS KELAMIN": "Jenis Kelamin",
    "USIA": "Usia",
    "Jenis hobi yang paling sering Anda lakukan adalah": "Jenis Hobi",
    "Seberapa sering Anda melakukan hobi dalam seminggu?": "Frekuensi per Minggu",
    "apakah  Hobi yang anda miliki membuat anda merasa lebih bahagia": "Hobi Membuat Bahagia",
    "apakah  Hobi membantu anda meningkatkan keterampilan atau kemampuan tertentu": "Hobi Meningkatkan Keterampilan",
    "apakah Hobi anda membantu mengurangi stres setelah melakukan aktivitas sehari-hari.": "Hobi Mengurangi Stres",
    "apakah anda merasa waktu luang  lebih bermanfaat ketika digunakan untuk melakukan hobi.": "Waktu Luang Lebih Bermanfaat",
    "Apakah anda sering berbagi atau melakukan hobi bersama teman atau komunitas.": "Berbagi Hobi dengan Komunitas",
}
df = df.rename(columns={k.strip(): v for k, v in rename_map.items()})

# Buang baris yang tidak punya data inti (mis. baris kosong)
df = df.dropna(subset=["Jenis Kelamin", "Usia", "Jenis Hobi"], how="all")

# ===================== SIDEBAR - FILTER =====================
st.sidebar.header("🔎 Filter Data")

gender_options = sorted(df["Jenis Kelamin"].dropna().unique().tolist())
usia_options = sorted(df["Usia"].dropna().unique().tolist())
hobi_options = sorted(df["Jenis Hobi"].dropna().unique().tolist())

selected_gender = st.sidebar.multiselect("Jenis Kelamin", gender_options, default=gender_options)
selected_usia = st.sidebar.multiselect("Kelompok Usia", usia_options, default=usia_options)
selected_hobi = st.sidebar.multiselect("Jenis Hobi", hobi_options, default=hobi_options)

filtered_df = df[
    (df["Jenis Kelamin"].isin(selected_gender)) &
    (df["Usia"].isin(selected_usia)) &
    (df["Jenis Hobi"].isin(selected_hobi))
]

# ===================== HEADER =====================
st.title("🎯 Dashboard Survei Seputar Hobi")
st.markdown("Visualisasi data hasil survei mengenai hobi, frekuensi, dan manfaatnya bagi responden.")
st.divider()

# ===================== METRIC RINGKASAN =====================
col1, col2, col3, col4 = st.columns(4)
col1.metric("Total Responden", len(filtered_df))
col2.metric("Jenis Hobi Terbanyak", 
            filtered_df["Jenis Hobi"].mode()[0] if not filtered_df["Jenis Hobi"].mode().empty else "-")
col3.metric("Kelompok Usia Dominan", 
            filtered_df["Usia"].mode()[0] if not filtered_df["Usia"].mode().empty else "-")
persen_bahagia = (filtered_df["Hobi Membuat Bahagia"] == "YA").mean() * 100 if len(filtered_df) > 0 else 0
col4.metric("Hobi Membuat Bahagia", f"{persen_bahagia:.0f}%")

st.divider()

# ===================== VISUALISASI =====================
tab1, tab2, tab3 = st.tabs(["📊 Demografi", "🎨 Hobi & Frekuensi", "💡 Manfaat Hobi"])

with tab1:
    c1, c2 = st.columns(2)
    with c1:
        st.subheader("Distribusi Jenis Kelamin")
        gender_count = filtered_df["Jenis Kelamin"].value_counts().reset_index()
        gender_count.columns = ["Jenis Kelamin", "Jumlah"]
        fig = px.pie(gender_count, names="Jenis Kelamin", values="Jumlah", hole=0.4)
        st.plotly_chart(fig, use_container_width=True)

    with c2:
        st.subheader("Distribusi Kelompok Usia")
        usia_count = filtered_df["Usia"].value_counts().reset_index()
        usia_count.columns = ["Usia", "Jumlah"]
        fig = px.bar(usia_count, x="Usia", y="Jumlah", color="Usia", text="Jumlah")
        st.plotly_chart(fig, use_container_width=True)

with tab2:
    c1, c2 = st.columns(2)
    with c1:
        st.subheader("Jenis Hobi Paling Sering Dilakukan")
        hobi_count = filtered_df["Jenis Hobi"].value_counts().reset_index()
        hobi_count.columns = ["Jenis Hobi", "Jumlah"]
        fig = px.bar(hobi_count, x="Jenis Hobi", y="Jumlah", color="Jenis Hobi", text="Jumlah")
        st.plotly_chart(fig, use_container_width=True)

    with c2:
        st.subheader("Frekuensi Hobi per Minggu")
        freq_count = filtered_df["Frekuensi per Minggu"].value_counts().reset_index()
        freq_count.columns = ["Frekuensi", "Jumlah"]
        fig = px.pie(freq_count, names="Frekuensi", values="Jumlah", hole=0.4)
        st.plotly_chart(fig, use_container_width=True)

    st.subheader("Jenis Hobi berdasarkan Jenis Kelamin")
    cross = filtered_df.groupby(["Jenis Hobi", "Jenis Kelamin"]).size().reset_index(name="Jumlah")
    fig = px.bar(cross, x="Jenis Hobi", y="Jumlah", color="Jenis Kelamin", barmode="group")
    st.plotly_chart(fig, use_container_width=True)

with tab3:
    manfaat_cols = [
        "Hobi Membuat Bahagia",
        "Hobi Meningkatkan Keterampilan",
        "Hobi Mengurangi Stres",
        "Waktu Luang Lebih Bermanfaat",
        "Berbagi Hobi dengan Komunitas",
    ]
    manfaat_cols = [c for c in manfaat_cols if c in filtered_df.columns]

    rekap = []
    for col in manfaat_cols:
        ya = (filtered_df[col] == "YA").sum()
        tidak = (filtered_df[col] == "TIDAK").sum()
        rekap.append({"Pertanyaan": col, "YA": ya, "TIDAK": tidak})
    rekap_df = pd.DataFrame(rekap)

    st.subheader("Rekap Jawaban Manfaat Hobi (YA vs TIDAK)")
    rekap_melt = rekap_df.melt(id_vars="Pertanyaan", var_name="Jawaban", value_name="Jumlah")
    fig = px.bar(rekap_melt, x="Jumlah", y="Pertanyaan", color="Jawaban", orientation="h", barmode="group")
    st.plotly_chart(fig, use_container_width=True)

st.divider()

# ===================== DATA MENTAH =====================
with st.expander("📄 Lihat Data Mentah"):
    st.dataframe(filtered_df, use_container_width=True)
    csv = filtered_df.to_csv(index=False).encode("utf-8")
    st.download_button("Unduh Data (CSV)", csv, "data_hobi_filtered.csv", "text/csv")
