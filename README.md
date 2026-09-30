import streamlit as st
import pandas as pd

# ==============================
# CẤU HÌNH TRANG
# ==============================

st.set_page_config(
    page_title="Tính lãi tiền gửi tiết kiệm",
    page_icon="💰",
    layout="centered"
)

# ==============================
# CSS
# ==============================

st.markdown("""
<style>
    .main-title {
        text-align: center;
        font-size: 32px;
        font-weight: bold;
        margin-bottom: 10px;
    }

    .sub-title {
        text-align: center;
        color: #666;
        margin-bottom: 30px;
    }

    .result-box {
        padding: 20px;
        border-radius: 12px;
        background-color: #f5f7fa;
        margin-bottom: 15px;
    }

    .result-title {
        font-size: 16px;
        color: #555;
    }

    .result-value {
        font-size: 24px;
        font-weight: bold;
    }
</style>
""", unsafe_allow_html=True)

# ==============================
# TIÊU ĐỀ
# ==============================

st.markdown(
    '<div class="main-title">💰 MÁY TÍNH LÃI TIỀN GỬI TIẾT KIỆM</div>',
    unsafe_allow_html=True
)

st.markdown(
    '<div class="sub-title">Tính lãi đơn và lãi kép theo kỳ hạn gửi tiết kiệm</div>',
    unsafe_allow_html=True
)

# ==============================
# NHẬP THÔNG TIN
# ==============================

st.subheader("📋 Thông tin tiền gửi")

tien_gui = st.number_input(
    "💵 Số tiền gửi (VNĐ)",
    min_value=0.0,
    value=100_000_000.0,
    step=1_000_000.0,
    format="%.0f"
)

ky_han = st.number_input(
    "📅 Kỳ hạn (tháng)",
    min_value=1,
    max_value=600,
    value=12,
    step=1
)

lai_suat = st.number_input(
    "📈 Lãi suất (%/năm)",
    min_value=0.0,
    max_value=100.0,
    value=6.0,
    step=0.1
)

loai_lai = st.selectbox(
    "🔢 Hình thức tính lãi",
    [
        "Lãi đơn",
        "Lãi kép"
    ]
)

hinh_thuc_nhan = st.selectbox(
    "💳 Hình thức nhận lãi",
    [
        "Lãnh lãi hàng tháng",
        "Lãnh lãi hàng quý",
        "Lãnh lãi cuối kỳ"
    ]
)

st.divider()

# ==============================
# HÀM ĐỊNH DẠNG TIỀN
# ==============================

def format_money(value):
    return f"{value:,.0f} VNĐ"


# ==============================
# TÍNH TOÁN
# ==============================

if tien_gui > 0 and ky_han > 0:

    r_nam = lai_suat / 100
    r_thang = r_nam / 12

    # --------------------------------
    # XÁC ĐỊNH SỐ KỲ NHẬN LÃI
    # --------------------------------

    if hinh_thuc_nhan == "Lãnh lãi hàng tháng":
        so_ky = ky_han
        so_thang_moi_ky = 1

    elif hinh_thuc_nhan == "Lãnh lãi hàng quý":
        so_ky = ky_han // 3
        so_thang_moi_ky = 3

        if ky_han % 3 != 0:
            so_ky += 1

    else:
        so_ky = 1
        so_thang_moi_ky = ky_han

    # ==============================
    # LÃI ĐƠN
    # ==============================if loai_lai == "Lãi đơn":

        tong_lai = tien_gui * r_nam * (ky_han / 12)

        lai_moi_thang = tien_gui * r_thang

        if hinh_thuc_nhan == "Lãnh lãi hàng tháng":

            lai_dinh_ky = lai_moi_thang

        elif hinh_thuc_nhan == "Lãnh lãi hàng quý":

            lai_dinh_ky = lai_moi_thang * 3

        else:

            lai_dinh_ky = tong_lai

        tong_goc_lai = tien_gui + tong_lai

        # --------------------------
        # BẢNG CHI TIẾT
        # --------------------------

        data = []

        for ky in range(1, so_ky + 1):

            if hinh_thuc_nhan == "Lãnh lãi hàng tháng":
                thang = ky
                lai_ky = lai_moi_thang

            elif hinh_thuc_nhan == "Lãnh lãi hàng quý":
                thang = min(ky * 3, ky_han)

                if ky == so_ky and ky_han % 3 != 0:
                    so_thang = ky_han - (ky - 1) * 3
                    lai_ky = tien_gui * r_thang * so_thang
                else:
                    lai_ky = tien_gui * r_thang * 3

            else:
                thang = ky_han
                lai_ky = tong_lai

            data.append({
                "Kỳ": ky,
                "Tháng": thang,
                "Tiền lãi kỳ này": lai_ky,
                "Tổng lãi lũy kế": sum(
                    x["Tiền lãi kỳ này"] for x in data
                ) + lai_ky
            })

    # ==============================
    # LÃI KÉP
    # ==============================

    else:

        # Trường hợp lãnh lãi cuối kỳ:
        # Lãi nhập vào gốc và tính lãi kép theo tháng.

        if hinh_thuc_nhan == "Lãnh lãi cuối kỳ":

            so_thang = ky_han

            tong_goc_lai = tien_gui * ((1 + r_thang) ** so_thang)

            tong_lai = tong_goc_lai - tien_gui

            lai_dinh_ky = tong_lai

            data = []

            so_du = tien_gui

            for thang in range(1, so_thang + 1):

                lai_thang = so_du * r_thang
                so_du += lai_thang

                data.append({
                    "Kỳ": thang,
                    "Tháng": thang,
                    "Tiền lãi kỳ này": lai_thang,
                    "Tổng lãi lũy kế": so_du - tien_gui
                })

        # --------------------------
        # LÃNH LÃI THÁNG
        # --------------------------

        elif hinh_thuc_nhan == "Lãnh lãi hàng tháng":

            # Khi lãi được lãnh ra ngoài hàng tháng,
            # tiền lãi không nhập vào gốc.
            # Vì vậy về bản chất không tạo ra lãi kép.

            lai_moi_thang = tien_gui * r_thang

            tong_lai = lai_moi_thang * ky_han

            tong_goc_lai = tien_gui + tong_lai

            lai_dinh_ky = lai_moi_thang

            data = []

            tong_luy_ke = 0

            for thang in range(1, ky_han + 1):

                tong_luy_ke += lai_moi_thangdata.append({
                    "Kỳ": thang,
                    "Tháng": thang,
                    "Tiền lãi kỳ này": lai_moi_thang,
                    "Tổng lãi lũy kế": tong_luy_ke
                })

        # --------------------------
        # LÃNH LÃI QUÝ
        # --------------------------

        else:

            # Lãi nhập vào gốc mỗi quý
            so_quy = ky_han // 3
            thang_du = ky_han % 3

            so_du = tien_gui
            data = []

            tong_lai = 0

            for quy in range(1, so_quy + 1):

                lai_quy = so_du * (
                    (1 + r_thang) ** 3 - 1
                )

                so_du += lai_quy
                tong_lai += lai_quy

                data.append({
                    "Kỳ": quy,
                    "Tháng": quy * 3,
                    "Tiền lãi kỳ này": lai_quy,
                    "Tổng lãi lũy kế": tong_lai
                })

            # Nếu còn số tháng lẻ
            if thang_du > 0:

                lai_du = so_du * (
                    (1 + r_thang) ** thang_du - 1
                )

                so_du += lai_du
                tong_lai += lai_du

                data.append({
                    "Kỳ": so_quy + 1,
                    "Tháng": ky_han,
                    "Tiền lãi kỳ này": lai_du,
                    "Tổng lãi lũy kế": tong_lai
                })

            tong_goc_lai = so_du
            lai_dinh_ky = (
                data[0]["Tiền lãi kỳ này"]
                if len(data) > 0
                else 0
            )

    # ==============================
    # HIỂN THỊ KẾT QUẢ
    # ==============================

    st.subheader("📊 Kết quả")

    col1, col2 = st.columns(2)

    with col1:
        st.markdown(
            f"""
            <div class="result-box">
                <div class="result-title">💰 Tiền lãi định kỳ</div>
                <div class="result-value">
                    {format_money(lai_dinh_ky)}
                </div>
            </div>
            """,
            unsafe_allow_html=True
        )

    with col2:
        st.markdown(
            f"""
            <div class="result-box">
                <div class="result-title">📈 Tổng tiền lãi</div>
                <div class="result-value">
                    {format_money(tong_lai)}
                </div>
            </div>
            """,
            unsafe_allow_html=True
        )

    st.markdown(
        f"""
        <div class="result-box">
            <div class="result-title">🏦 Tổng số tiền gốc + lãi</div>
            <div class="result-value">
                {format_money(tong_goc_lai)}
            </div>
        </div>
        """,
        unsafe_allow_html=True
    )

    # ==============================
    # BẢNG CHI TIẾT
    # ============================== st.subheader("📑 Chi tiết theo từng kỳ")

    df = pd.DataFrame(data)

    if not df.empty:

        df["Tiền lãi kỳ này"] = df["Tiền lãi kỳ này"].apply(
            format_money
        )

        df["Tổng lãi lũy kế"] = df["Tổng lãi lũy kế"].apply(
            format_money
        )

        st.dataframe(
            df,
            use_container_width=True,
            hide_index=True
        )

    # ==============================
    # THÔNG TIN TÓM TẮT
    # ==============================

    st.divider()

    st.subheader("📝 Thông tin khoản gửi")

    col1, col2 = st.columns(2)

    with col1:
        st.write(f"**Số tiền gửi:** {format_money(tien_gui)}")
        st.write(f"**Kỳ hạn:** {ky_han} tháng")
        st.write(f"**Lãi suất:** {lai_suat:.2f}%/năm")

    with col2:
        st.write(f"**Hình thức:** {loai_lai}")
        st.write(f"**Nhận lãi:** {hinh_thuc_nhan}")

else:
    st.warning("Vui lòng nhập số tiền gửi và kỳ hạn hợp lệ.")
