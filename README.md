# appquantrasuaimport streamlit as st
import pandas as pd
from datetime import datetime

# ---------------------------------------------------------
# 1. CẤU HÌNH TRANG & GIAO DIỆN
# ---------------------------------------------------------
st.set_page_config(
    page_title="Quản Lý Hóa Đơn Trà Sữa",
    page_icon="🧋",
    layout="wide"
)

# Thêm CSS tùy chỉnh cho giao diện đẹp hơn
st.markdown("""
    <style>
    .main-title {
        text-align: center;
        color: #FF4B4B;
        font-family: 'Helvetica Neue', sans-serif;
        font-weight: bold;
        margin-bottom: 20px;
    }
    .bill-box {
        background-color: #F0F2F6;
        padding: 20px;
        border-radius: 10px;
        border: 1px solid #D1D5DB;
        font-family: 'Courier New', Courier, monospace;
    }
    </style>
""", unsafe_allow_html=True)

st.markdown("<h1 class='main-title'>🧋 HỆ THỐNG TÍNH TIỀN TRÀ SỮA 🧋</h1>", unsafe_allow_html=True)

# ---------------------------------------------------------
# 2. KHỞI TẠO BẢNG GIÁ & DỮ LIỆU
# ---------------------------------------------------------
# Bảng giá Trà sữa
MENU_TRA_SUA = {
    "Trà sữa Truyền thống": 30000,
    "Trà sữa Trân châu Đường đen": 35000,
    "Trà sữa Matcha": 38000,
    "Trà sữa Thái Xanh / Thái Đỏ": 32000,
    "Trà Oolong Sữa": 38000,
    "Trà Trái Cây Nhiệt Đới": 35000,
    "Trà Đào Cam Sả": 35000
}

# Phụ thu Size
MENU_SIZE = {
    "Size S": 0,
    "Size M": 5000,
    "Size L": 10000
}

# Bảng giá Topping
MENU_TOPPING = {
    "Trân châu đen": 5000,
    "Trân châu trắng": 7000,
    "Thạch trái cây": 5000,
    "Pudding trứng": 8000,
    "Kem Cheese": 10000
}

MUC_DUONG = ["100% (Bình thường)", "70%", "50%", "30%", "0% (Không đường)"]
MUC_DA = ["100% (Bình thường)", "70%", "50%", "30%", "0% (Không đá)"]

# Khởi tạo giỏ hàng trong session_state
if "gio_hang" not in st.session_state:
    st.session_state.gio_hang = []

# ---------------------------------------------------------
# 3. CHIA BỐ CỤC CỘT (NHẬP LIỆU & HÓA ĐƠN)
# ---------------------------------------------------------
col1, col2 = st.columns([1.2, 1])

# =========================================================
# CỘT 1: NHẬP THÔNG TIN KHÁCH HÀNG & MÓN ĂN
# =========================================================
with col1:
    st.subheader("📋 Thông tin đơn hàng")
    
    # 1. Thông tin khách hàng
    ten_khach = st.text_input("👤 Tên khách hàng / Số bàn:", value="Khách Lẻ")
    
    st.divider()
    st.subheader("➕ Thêm món vào hóa đơn")
    
    # Form chọn món
    loai_ts = st.selectbox("🥤 Chọn loại trà sữa / nước uống:", list(MENU_TRA_SUA.keys()))
    
    c_size, c_so_luong = st.columns(2)
    with c_size:
        size = st.selectbox("📏 Chọn Size:", list(MENU_SIZE.keys()), index=1) # Mặc định Size M
    with c_so_luong:
        so_luong = st.number_input("🔢 Số lượng:", min_value=1, max_value=50, value=1, step=1)

    c_duong, c_da = st.columns(2)
    with c_duong:
        duong = st.selectbox("🍬 Mức độ đường:", MUC_DUONG)
    with c_da:
        da = st.selectbox("🧊 Mức độ đá:", MUC_DA)

    toppings = st.multiselect("🧆 Chọn Topping thêm (có thể chọn nhiều):", list(MENU_TOPPING.keys()))

    # Tính giá tiền món hiện tại
    gia_goc = MENU_TRA_SUA[loai_ts]
    gia_size = MENU_SIZE[size]
    gia_topping = sum(MENU_TOPPING[t] for t in toppings)
    don_gia = gia_goc + gia_size + gia_topping
    thanh_tien_mon = don_gia * so_luong

    st.markdown(f"**Đơn giá 1 ly:** `{don_gia:,} VNĐ` | **Thành tiền:** `{thanh_tien_mon:,} VNĐ`")

    # Nút thêm món vào giỏ
    if st.button("➕ Thêm vào hóa đơn", type="primary", use_container_width=True):
        mon_moi = {
            "ten_mon": loai_ts,
            "size": size,
            "duong": duong,
            "da": da,
            "topping": ", ".join(toppings) if toppings else "Không",
            "don_gia": don_gia,
            "so_luong": so_luong,
            "thanh_tien": thanh_tien_mon
        }
        st.session_state.gio_hang.append(mon_moi)
        st.toast(f"Đã thêm {so_luong}x {loai_ts} vào đơn hàng!", icon="✅")

# =========================================================
# CỘT 2: CHI TIẾT GIỎ HÀNG & HIỂN THỊ HÓA ĐƠN
# =========================================================
with col2:
    st.subheader("🛒 Món đã chọn")

    if not st.session_state.gio_hang:
        st.info("Giỏ hàng đang trống. Vui lòng chọn món và bấm 'Thêm vào hóa đơn'.")
    else:
        # Hiển thị danh sách dạng bảng
        df = pd.DataFrame(st.session_state.gio_hang)
        
        # Chọn các cột để hiển thị ngắn gọn trên giao diện
        df_display = df[["ten_mon", "size", "so_luong", "thanh_tien"]].copy()
        df_display.columns = ["Món", "Size", "SL", "Thành tiền (VNĐ)"]
        df_display["Thành tiền (VNĐ)"] = df_display["Thành tiền (VNĐ)"].map("{:,}".format)
        
        st.dataframe(df_display, use_container_width=True, hide_index=True)

        # Tính tổng số tiền
        tong_tien = sum(item["thanh_tien"] for item in st.session_state.gio_hang)
        st.markdown(f"### 💰 **Tổng tiền: {tong_tien:,} VNĐ**")

        c_huy, c_thanhtoan = st.columns(2)
        with c_huy:
            if st.button("🗑️ Xóa tất cả", use_container_width=True):
                st.session_state.gio_hang = []
                st.rerun()

# ---------------------------------------------------------
# 4. XUẤT HÓA ĐƠN KHI BẤM THANH TOÁN
# ---------------------------------------------------------
st.divider()

if st.session_state.gio_hang:
    if st.button("🧾 XUẤT HÓA ĐƠN THANH TOÁN", type="primary", use_container_width=True):
        now = datetime.now().strftime("%d/%m/%Y %H:%M:%S")
        tong_tien = sum(item["thanh_tien"] for item in st.session_state.gio_hang)

        # Tạo nội dung Text Hóa Đơn
        hoa_don_txt = "==========================================\n"
        hoa_don_txt += "         🧋 QUÁN TRÀ SỮA BOBA 🧋\n"
        hoa_don_txt += "    Địa chỉ: 123 Đường ABC, Quận 1, TP.HCM\n"
        hoa_don_txt += "    Hotline: 0909 123 456\n"
        hoa_don_txt += "==========================================\n"
        hoa_don_txt += f"Thời gian : {now}\n"
        hoa_don_txt += f"Khách hàng: {ten_khach}\n"
        hoa_don_txt += "------------------------------------------\n"
        
        for idx, item in enumerate(st.session_state.gio_hang, 1):
            hoa_don_txt += f"{idx}. {item['ten_mon']} ({item['size']})\n"
            hoa_don_txt += f"   - Tùy chọn : Đường {item['duong']} | Đá {item['da']}\n"
            hoa_don_txt += f"   - Topping  : {item['topping']}\n"
            hoa_don_txt += f"   - SL x Giá : {item['so_luong']} x {item['don_gia']:,} VNĐ\n"
            hoa_don_txt += f"   => Tiền    : {item['thanh_tien']:,} VNĐ\n"
            hoa_don_txt += "..........................................\n"
            
        hoa_don_txt += "------------------------------------------\n"
        hoa_don_txt += f"TỔNG CỘNG THANH TOÁN: {tong_tien:,} VNĐ\n"
        hoa_don_txt += "==========================================\n"
        hoa_don_txt += "   CẢM ƠN QUÝ KHÁCH VÀ HẸN GẶP LẠI!\n"
        hoa_don_txt += "==========================================\n"

        # Hiển thị hóa đơn ra màn hình
        st.success("✅ Thanh toán thành công! Dưới đây là hóa đơn của bạn:")
        st.code(hoa_don_txt, language="text")

        # Nút tải file hóa đơn về máy (.txt)
        file_name_bill = f"HoaDon_{ten_khach.replace(' ', '_')}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
        
        st.download_button(
            label="📥 Tải hóa đơn dạng File Text (.txt)",
            data=hoa_don_txt,
            file_name=file_name_bill,
            mime="text/plain",
            use_container_width=True
        )
