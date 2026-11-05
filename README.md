#include <stdio.h>
#include <errno.h>
#include <string.h>
 
#define MAX_NV 100
#define MA_NV_SIZE 20
#define FILE_NAME "nhanvien.dat"
#define NGAY_CONG_CHUAN 26
// note
struct FileHeader {
    char magic[8];
    unsigned int version;
    unsigned int soLuong;
    int thang;
    int nam;
};
//note
struct DT07LegacyHeader {
    char magic[8];
    unsigned int version;
    unsigned int soLuong;
};

// thông tin 1 nhân viên
struct NhanVien {
    char maNV[MA_NV_SIZE];
    char ten[50];
    int tuoi;
    char gioiTinh[20];
    char P_ban[40];
    char C_vu[40];
    int ngayCong;
    double luong;
    double phuCap;
    double baoHiem;
    double thueTNCN;
};
// void xóa xuống dòng
void xoa_Xuongdong(){
    int kyTu;
    while ((kyTu = getchar()) != '\n' && kyTu != EOF );
    }
// void đếm  số lượng nhân viên
int demSoLuongNhanVien(const struct NhanVien *ds) {
    int dem = 0;

    for (int i = 0; i < MAX_NV; i++) {
        if (ds[i].maNV[0] != '\0') {
            dem++;
        }
    }

    return dem;
}
// Ktra tuổi, lương
int tuoiHopLe(int tuoi) {
    return tuoi >= 0 && tuoi <= 60;
}

int luongHopLe(double luong) {
    return luong >= 0;
}

// Nhập thuế suất minh họa riêng cho nhân viên, theo phần trăm từ 0 đến 100.
int nhapThueTNCN(double *thueTNCN) {
    while (1) {
        printf("Thue TNCN (0-100%%): ");
        int ketQua = scanf("%lf", thueTNCN);

        if (ketQua == EOF) {
            printf("Khong con du lieu de doc thue TNCN.\n");
            return 0;
        }
        if (ketQua == 1 && *thueTNCN >= 0.0 && *thueTNCN <= 100.0) {
            xoa_Xuongdong();
            return 1;
        }

        xoa_Xuongdong();
        printf("Thue TNCN phai la so tu 0 den 100.\n");
    }
}

// Nhập thông tin và thêm nhân viên .
int themNhanvien(struct NhanVien ds[], int *soLuong) {
    struct NhanVien nvMoi = {0};
    int ketQua;

    if (*soLuong >= MAX_NV) {
        printf("Danh sach da day, khong the them nhan vien.\n");
        return 0;
    }

    while (1) {
        int trungMa = 0;
        char maNhap[MA_NV_SIZE + 1];

        printf("Ma nhan vien: ");
        ketQua = scanf("%20s", maNhap); // ketQua = 1 khi sacnf thành công
        xoa_Xuongdong();

        if (ketQua == EOF) { // nhập dữ liệu bị đóng hay kết thúc -> chưa nhập maNV
            printf("Khong con du lieu de doc ma nhan vien.\n");
            return 0;
        }
        if (ketQua != 1) {
            printf("Khong doc duoc ma nhan vien.\n");
            continue;
        }
        if (strlen(maNhap) >= MA_NV_SIZE) {
            printf("Ma nhan vien toi da %d ky tu.\n", MA_NV_SIZE - 1);
            continue;
        }
        strcpy(nvMoi.maNV, maNhap);

        for (int i = 0; i < *soLuong; i++) {
            if (strcmp(ds[i].maNV, nvMoi.maNV) == 0) {
                trungMa = 1;
                break;
            }
        }
        if (!trungMa) { // !trungMa là phủ định của trungMa, ( 0 -> 1 : chạy if; 1 -> 0 : không chạy if)
            break;
        }
        printf("Ma nhan vien da ton tai, vui long nhap ma khac.\n");
    }

    printf("Ho va ten: ");
    if (fgets(nvMoi.ten, sizeof(nvMoi.ten), stdin) == NULL) {
        printf("Khong doc duoc ho va ten.\n");
        return 0;
    }
    nvMoi.ten[strcspn(nvMoi.ten, "\n")] = '\0';

    // Nhập và kiểm tra tuổi
    while (1) {
        printf("Tuoi (0-60): ");
        if (scanf("%d", &nvMoi.tuoi) != 1) {
            printf("Tuoi phai la so nguyen.\n");
            xoa_Xuongdong();
            continue;
        }
        xoa_Xuongdong();
        if (tuoiHopLe(nvMoi.tuoi)) {
            break;
        }
        printf("Tuoi phai tu 0 den 60.\n");
    }
    
    // them giới tính
    printf("Gioi tinh: ");
    if (fgets(nvMoi.gioiTinh, sizeof(nvMoi.gioiTinh), stdin) == NULL) {
        printf("Khong doc duoc gioi tinh.\n");
        return 0;
    }
    nvMoi.gioiTinh[strcspn(nvMoi.gioiTinh, "\n")] = '\0';
    
    // thêm phòng ban
    printf("Phong ban: ");
    if (fgets(nvMoi.P_ban, sizeof(nvMoi.P_ban), stdin) == NULL) {
        printf("Khong doc duoc phong ban.\n");
        return 0;
    }
    nvMoi.P_ban[strcspn(nvMoi.P_ban, "\n")] = '\0';
    
    //thêm chức vụ
    printf("Chuc vu: ");
    if (fgets(nvMoi.C_vu, sizeof(nvMoi.C_vu), stdin) == NULL) {
        printf("Khong doc duoc chuc vu.\n");
        return 0;
    }
    nvMoi.C_vu[strcspn(nvMoi.C_vu, "\n")] = '\0';

    // Nhập số ngày chấm công
    while (1) {
        printf("So ngay cham cong: ");
        if (scanf("%d", &nvMoi.ngayCong) != 1) {
            printf("So ngay cham cong phai la so nguyen khong am.\n");
            xoa_Xuongdong();
            continue;
        }
        xoa_Xuongdong();
        if (nvMoi.ngayCong >= 0) {
            break;
        }
        printf("So ngay cham cong khong duoc am.\n");
    }
    
    // Nhập và kiểm tra lương
    while (1) {
        printf("Luong: ");
        if (scanf("%lf", &nvMoi.luong) != 1) {
            printf("Luong phai la mot so.\n");
            xoa_Xuongdong();
            continue;
        }
        xoa_Xuongdong();
        if (luongHopLe(nvMoi.luong)) {
            break;
        }
        printf("Luong phai khong am.\n");
    }

    if (!nhapThueTNCN(&nvMoi.thueTNCN)) {
        return 0;
    }

    ds[*soLuong] = nvMoi;
    (*soLuong)++;
    printf("Da them nhan vien thanh cong.\n");
    return 1;
}
// xuất thông tin 1 nhân viên
void xuatMotNhanVien(const struct NhanVien *nv) {
    printf("Ma NV: %-19s | Ten: %-20s | Tuoi: %d | G_tinh: %-8s | "
           "C_vu: %-15s | P_ban: %-15s |\n",
           nv->maNV,
           nv->ten,
           nv->tuoi,
           nv->gioiTinh,
           nv->C_vu,
           nv->P_ban);
}
// xuất danh sách nhân viên
void xuatDanhsach(const struct NhanVien *ds, int soLuong){
    if (soLuong <= 0) {
        printf("Danh sach nhan vien rong.\n");
        return;
    }

    for (int i = 0; i < soLuong; i++) {
        xuatMotNhanVien(&ds[i]);
    }
}
// lương cao nhất
void luong_Best(const struct NhanVien *nv, int soLuong) { // *nv là con trỏ truy cập và ds
        if (soLuong <= 0) {
            printf("Danh sach nhan vien rong.\n");
            return;
        }
    
        double maxLuong = nv[0].luong;
    
        // Tìm mức lương cao nhất
        for (int i = 1; i < soLuong; i++) {
            if (nv[i].luong > maxLuong) {
                maxLuong = nv[i].luong;
            }
        }
    
        printf("Luong cao nhat: %.0f\n", maxLuong);
    
        // In nhân viên có mức lương cao nhất
        for (int i = 0; i < soLuong; i++) {
            if (nv[i].luong == maxLuong) {
                printf("Ma NV: %s | Ten: %s | Luong: %.0f\n",
                       nv[i].maNV, nv[i].ten, nv[i].luong);
            }
        }
    }

// sắp xếp thứ tự theo lương
void sapXepLuongGiamDan(struct NhanVien *ds, int soLuong) {
    for (int i = 0; i < soLuong - 1; i++) { // do số lượng tính từ 1
        for (int j = 0; j < soLuong - 1 - i; j++) {
            if (ds[j].luong < ds[j + 1].luong) {
                struct NhanVien tam = ds[j]; // tam khai báo biểu struct NhanVien -> chứa đc thông tin của 1 nhânn viên
                ds[j] = ds[j + 1];
                ds[j + 1] = tam;
            }
        }
    }
}
// tìm theo mã nhân viên
void timNhanVien(struct NhanVien *ds, int soLuong, const char *maCanTim) {
    for (int i = 0; i < soLuong; i++) {
        if (strcmp(ds[i].maNV, maCanTim) == 0) {
            printf("Tim thay nhan vien:\n");
            xuatMotNhanVien(&ds[i]);
            return;
        }
    }

    printf("Khong tim thay nhan vien co ma %s.\n", maCanTim);
}

// xóa nhân viên theo mã
int xoaNhanVien(struct NhanVien ds[], int *soLuong) {
    char maNhap[MA_NV_SIZE + 1];
    char maCanXoa[MA_NV_SIZE];
    char xacNhan;
    int viTri = -1;

    if (*soLuong <= 0) {
        printf("Danh sach nhan vien rong.\n");
        return 0;
    }

    printf("Nhap ma nhan vien can xoa: ");
    if (scanf("%20s", maNhap) != 1) {
        xoa_Xuongdong();
        printf("Khong doc duoc ma nhan vien.\n");
        return 0;
    }
    xoa_Xuongdong();
    if (strlen(maNhap) >= MA_NV_SIZE) {
        printf("Ma nhan vien toi da %d ky tu.\n", MA_NV_SIZE - 1);
        return 0;
    }
    strcpy(maCanXoa, maNhap); // strcpy() chép chuỗi trong c chép maNhap sang maCanxoa

    for (int i = 0; i < *soLuong; i++) {
        if (strcmp(ds[i].maNV, maCanXoa) == 0) {
            viTri = i;
            break;
        }
    }

    if (viTri == -1) {
        printf("Khong tim thay nhan vien co ma %s.\n", maCanXoa);
        return 0;
    }

    printf("Xac nhan xoa nhan vien %s? (y/n): ", maCanXoa);
    if (scanf(" %c", &xacNhan) != 1) {
        xoa_Xuongdong();
        printf("Khong doc duoc lua chon xac nhan.\n");
        return 0;
    }
    xoa_Xuongdong();

    if (xacNhan != 'y' && xacNhan != 'Y') {
        printf("Da huy thao tac xoa.\n");
        return 0;
    }

    for (int i = viTri; i < *soLuong - 1; i++) {
        ds[i] = ds[i + 1];
    }

    (*soLuong)--;
    memset(&ds[*soLuong], 0, sizeof(ds[*soLuong]));
    printf("Da xoa nhan vien %s.\n", maCanXoa);
    return 1;
}

// sửa thông tin nhân viên theo mã
void suaNhanVien(struct NhanVien *ds, int soLuong, const char *maCanSua) {
    for (int i = 0; i < soLuong; i++) {
        if (strcmp(ds[i].maNV, maCanSua) != 0) {
            continue;
        }

        printf("Nhap ho va ten moi: ");
        if (fgets(ds[i].ten, sizeof(ds[i].ten), stdin) == NULL) {
            printf("Khong doc duoc ho va ten.\n");
            return;
        }
        ds[i].ten[strcspn(ds[i].ten, "\n")] = '\0';

        while (1) {
            printf("Nhap tuoi moi (0-60): ");
            if (scanf("%d", &ds[i].tuoi) != 1) {
                printf("Tuoi phai la so nguyen.\n");
                xoa_Xuongdong();
                continue;
            }
            xoa_Xuongdong();
            if (tuoiHopLe(ds[i].tuoi)) {
                break;
            }
            printf("Tuoi phai tu 0 den 60.\n");
        }

        printf("Nhap gioi tinh moi: ");
        if (fgets(ds[i].gioiTinh, sizeof(ds[i].gioiTinh), stdin) == NULL) {
            printf("Khong doc duoc gioi tinh.\n");
            return;
        }
        ds[i].gioiTinh[strcspn(ds[i].gioiTinh, "\n")] = '\0';

        printf("Nhap phong ban moi: ");
        if (fgets(ds[i].P_ban, sizeof(ds[i].P_ban), stdin) == NULL) {
            printf("Khong doc duoc phong ban.\n");
            return;
        }
        ds[i].P_ban[strcspn(ds[i].P_ban, "\n")] = '\0';

        printf("Nhap chuc vu moi: ");
        if (fgets(ds[i].C_vu, sizeof(ds[i].C_vu), stdin) == NULL) {
            printf("Khong doc duoc chuc vu.\n");
            return;
        }
        ds[i].C_vu[strcspn(ds[i].C_vu, "\n")] = '\0';

        while (1) {
            printf("Nhap luong co ban moi: ");
            if (scanf("%lf", &ds[i].luong) != 1) {
                printf("Luong phai la mot so.\n");
                xoa_Xuongdong();
                continue;
            }
            xoa_Xuongdong();
            if (luongHopLe(ds[i].luong)) {
                break;
            }
            printf("Luong phai khong am.\n");
        }

        if (!nhapThueTNCN(&ds[i].thueTNCN)) {
            return;
        }

        printf("Da cap nhat thong tin nhan vien.\n");
        return;
    }

    printf("Khong tim thay nhan vien co ma %s.\n", maCanSua);
}

// cập nhật số ngày công theo mã nhân viên
void capNhatChamCong(struct NhanVien *ds, int soLuong, const char *maCanCapNhat) {
    for (int i = 0; i < soLuong; i++) {
        if (strcmp(ds[i].maNV, maCanCapNhat) != 0) {
            continue;
        }

        int ngayCongMoi;
        while (1) {
            printf("Nhap so ngay cong moi: ");
            int ketQua = scanf("%d", &ngayCongMoi);

            if (ketQua == EOF) {
                printf("Khong doc duoc so ngay cong.\n");
                return;
            }
            if (ketQua != 1) {
                printf("So ngay cong phai la so nguyen khong am.\n");
                xoa_Xuongdong();
                continue;
            }
            xoa_Xuongdong();

            if (ngayCongMoi >= 0) {
                ds[i].ngayCong = ngayCongMoi;
                printf("Da cap nhat ngay cong cho nhan vien %s.\n", maCanCapNhat);
                return;
            }

            printf("So ngay cong khong duoc am.\n");
        }
    }

    printf("Khong tim thay nhan vien co ma %s.\n", maCanCapNhat);
}

// Tính lương trước thuế theo ngày công và phụ cấp cố định hiện tại.
static double tinhLuongTruocThue(const struct NhanVien *nv, int ngayCongChuan) {
    return (nv->luong / ngayCongChuan) * nv->ngayCong + 2000000.0;
}

// ngayCongChuan phải lớn hơn 0; thueTNCN là thuế suất minh họa theo phần trăm (VD nhập 10 -> 10%)
double tinhLuong(const struct NhanVien *nv, int ngayCongChuan) {
    double luongTruocThue = tinhLuongTruocThue(nv, ngayCongChuan);
    double tienThue = luongTruocThue * nv->thueTNCN / 100.0;

    return luongTruocThue - tienThue;
}

// Hiển thị bảng lương và tổng số tiền thực lĩnh.
void inBangLuong(const struct NhanVien ds[], int soLuong, int ngayCongChuan) {
    double tongThucLinh = 0.0;

    if (soLuong <= 0) {
        printf("Danh sach nhan vien rong.\n");
        return;
    }
    if (ngayCongChuan <= 0) {
        printf("So ngay cong chuan phai lon hon 0.\n");
        return;
    }

    printf("\nBANG LUONG (ngay cong chuan: %d)\n", ngayCongChuan);
    printf("%-19s %-20s %10s %14s %14s %10s %12s %14s\n",
           "Ma NV", "Ho ten", "Ngay cong", "Luong co ban",
           "Luong truoc thue", "Thue suat", "Tien thue", "Thuc linh");
    printf("---------------------------------------------------------------------------------------------------------------\n");

    for (int i = 0; i < soLuong; i++) {
        double luongTruocThue = tinhLuongTruocThue(&ds[i], ngayCongChuan);
        double tienThue = luongTruocThue * ds[i].thueTNCN / 100.0;
        double thucLinh = tinhLuong(&ds[i], ngayCongChuan);

        printf("%-19s %-20.20s %10d %14.0f %14.0f %9.2f%% %12.0f %14.0f\n",
               ds[i].maNV, ds[i].ten, ds[i].ngayCong, ds[i].luong,
               luongTruocThue, ds[i].thueTNCN, tienThue, thucLinh);
        tongThucLinh += thucLinh;
    }

    printf("---------------------------------------------------------------------------------------------------------------\n");
    printf("Tong thuc linh: %.0f\n", tongThucLinh);
}

// Lưu danh sách vào tệp nhị phân.
//note
int luuDuLieu(const char *tenFile, const struct NhanVien ds[], int soLuong,
              int thang, int nam) {
    FILE *file;
    struct FileHeader header = {{0}, 2, (unsigned int)soLuong, thang, nam};

    if (soLuong < 0 || soLuong > MAX_NV) {
        fprintf(stderr, "So luong nhan vien khong hop le, khong the luu.\n");
        return 0;
    }
    if (thang < 1 || thang > 12 || nam < 2000 || nam > 9999) {
        fprintf(stderr, "Thang hoac nam khong hop le, khong the luu.\n");
        return 0;
    }

    memcpy(header.magic, "DT07DATA", sizeof(header.magic));
    file = fopen(tenFile, "wb");
    if (file == NULL) {
        perror("Khong mo duoc file de ghi");
        return 0;
    }

    if (fwrite(&header, sizeof(header), 1, file) != 1
        || (soLuong > 0
            && fwrite(ds, sizeof(ds[0]), (size_t)soLuong, file) != (size_t)soLuong)) {
        perror("Loi khi ghi file du lieu");
        fclose(file);
        return 0;
    }

    if (fclose(file) != 0) {
        perror("Loi khi dong file du lieu");
        return 0;
    }

    printf("Da luu %d nhan vien vao %s.\n", soLuong, tenFile);
    return 1;
}

// Nạp dữ liệu vào danh sách sau khi kiểm tra định dạng tệp.
//note
int docDuLieu(const char *tenFile, struct NhanVien ds[], int *soLuong,
              int *thang, int *nam) {
    FILE *file = fopen(tenFile, "rb");
    struct FileHeader header;
    struct DT07LegacyHeader legacyHeader;
    struct NhanVien danhSachTam[MAX_NV];
    unsigned int i;

    if (file == NULL) {
        if (errno != ENOENT) {
            perror("Khong mo duoc file de doc");
            return -1;
        }
        return 0;
    }

    if (fread(&legacyHeader, sizeof(legacyHeader), 1, file) != 1
        || memcmp(legacyHeader.magic, "DT07DATA", sizeof(legacyHeader.magic)) != 0
        || (legacyHeader.version != 1 && legacyHeader.version != 2)
        || legacyHeader.soLuong > MAX_NV) {
        fprintf(stderr, "File du lieu khong hop le hoac sai dinh dang.\n");
        fclose(file);
        return -1;
    }

    memset(&header, 0, sizeof(header));
    memcpy(header.magic, legacyHeader.magic, sizeof(header.magic));
    header.version = legacyHeader.version;
    header.soLuong = legacyHeader.soLuong;
    header.thang = 1;
    header.nam = 2026;
    if (header.version == 2
        && (fread(&header.thang, sizeof(header.thang), 1, file) != 1
            || fread(&header.nam, sizeof(header.nam), 1, file) != 1
            || header.thang < 1 || header.thang > 12
            || header.nam < 2000 || header.nam > 9999)) {
        fprintf(stderr, "Thang hoac nam trong file khong hop le.\n");
        fclose(file);
        return -1;
    }

    if (header.soLuong > 0
        && fread(danhSachTam, sizeof(danhSachTam[0]), header.soLuong, file) != header.soLuong) {
        fprintf(stderr, "File bi thieu du lieu nhan vien.\n");
        fclose(file);
        return -1;
    }

    for (i = 0; i < header.soLuong; i++) {
        if (memchr(danhSachTam[i].maNV, '\0', sizeof(danhSachTam[i].maNV)) == NULL
            || memchr(danhSachTam[i].ten, '\0', sizeof(danhSachTam[i].ten)) == NULL
            || memchr(danhSachTam[i].gioiTinh, '\0', sizeof(danhSachTam[i].gioiTinh)) == NULL
            || memchr(danhSachTam[i].P_ban, '\0', sizeof(danhSachTam[i].P_ban)) == NULL
            || memchr(danhSachTam[i].C_vu, '\0', sizeof(danhSachTam[i].C_vu)) == NULL) {
            fprintf(stderr, "Chuoi trong file du lieu khong hop le.\n");
            fclose(file);
            return -1;
        }
    }

    if (fclose(file) != 0) {
        perror("Loi khi dong file du lieu");
        return -1;
    }

    memcpy(ds, danhSachTam, header.soLuong * sizeof(ds[0]));
    *soLuong = (int)header.soLuong;
    *thang = header.thang;
    *nam = header.nam;
    return 1;
}
// note
static int doiThangChamCong(struct NhanVien ds[], int soLuong,
                            int *thang, int *nam) {
    int thangMoi;
    int namMoi;
    int ketQua;
    char xacNhan;

    printf("\nChuyen thang se xoa ngay cong cua tat ca nhan vien.\n");
    printf("Thong tin nhan vien va muc thue se duoc giu lai.\n");
    printf("Xac nhan chuyen thang? (y/n): ");
    ketQua = scanf(" %c", &xacNhan);
    xoa_Xuongdong();
    if (ketQua != 1 || (xacNhan != 'y' && xacNhan != 'Y')) {
        printf("Da huy chuyen thang.\n");
        return 0;
    }

    for (;;) {
        printf("Nhap thang moi (1-12): ");
        ketQua = scanf("%d", &thangMoi);
        xoa_Xuongdong();
        if (ketQua == EOF) {
            printf("Khong con du lieu de doc thang moi.\n");
            return 0;
        }
        if (ketQua == 1 && thangMoi >= 1 && thangMoi <= 12) {
            break;
        }
        printf("Thang phai la so nguyen tu 1 den 12.\n");
    }

    for (;;) {
        printf("Nhap nam moi (2000-9999): ");
        ketQua = scanf("%d", &namMoi);
        xoa_Xuongdong();
        if (ketQua == EOF) {
            printf("Khong con du lieu de doc nam moi.\n");
            return 0;
        }
        if (ketQua == 1 && namMoi >= 2000 && namMoi <= 9999) {
            break;
        }
        printf("Nam phai la so nguyen tu 2000 den 9999.\n");
    }

    if (thangMoi == *thang && namMoi == *nam) {
        printf("Thang moi trung voi thang hien tai; khong reset cham cong.\n");
        return 0;
    }

    for (int i = 0; i < soLuong; i++) {
        ds[i].ngayCong = 0;
    }
    *thang = thangMoi;
    *nam = namMoi;
    printf("Da chuyen sang thang %02d/%d va reset ngay cong.\n", *thang, *nam);
    return 1;
}

static int nhapThangNam(int *thangMoi, int *namMoi) {
    int ketQua;

    for (;;) {
        printf("Nhap thang (1-12): ");
        ketQua = scanf("%d", thangMoi);
        xoa_Xuongdong();
        if (ketQua == EOF) {
            printf("Khong con du lieu de doc thang.\n");
            return 0;
        }
        if (ketQua == 1 && *thangMoi >= 1 && *thangMoi <= 12) {
            break;
        }
        printf("Thang phai la so nguyen tu 1 den 12.\n");
    }

    for (;;) {
        printf("Nhap nam (2000-9999): ");
        ketQua = scanf("%d", namMoi);
        xoa_Xuongdong();
        if (ketQua == EOF) {
            printf("Khong con du lieu de doc nam.\n");
            return 0;
        }
        if (ketQua == 1 && *namMoi >= 2000 && *namMoi <= 9999) {
            return 1;
        }
        printf("Nam phai la so nguyen tu 2000 den 9999.\n");
    }
}

static int resetToanBoDuLieu(struct NhanVien ds[], int *soLuong,
                             int *thang, int *nam) {
    struct NhanVien danhSachRong[MAX_NV] = {0};
    int thangMoi;
    int namMoi;
    char xacNhan;

    printf("\nCANH BAO: thao tac nay se xoa toan bo ho so nhan vien va cham cong.\n");
    printf("Du lieu luu trong file cung se duoc thay bang danh sach rong.\n");
    printf("Nhap y de xac nhan, n de huy: ");
    if (scanf(" %c", &xacNhan) != 1) {
        xoa_Xuongdong();
        printf("Khong doc duoc lua chon xac nhan.\n");
        return 0;
    }
    xoa_Xuongdong();
    if (xacNhan != 'y' && xacNhan != 'Y') {
        printf("Da huy reset du lieu.\n");
        return 0;
    }

    if (!nhapThangNam(&thangMoi, &namMoi)) {
        return 0;
    }

    if (!luuDuLieu(FILE_NAME, danhSachRong, 0, thangMoi, namMoi)) {
        printf("Khong reset du lieu vi khong luu duoc file.\n");
        return 0;
    }

    memset(ds, 0, MAX_NV * sizeof(ds[0]));
    *soLuong = 0;
    *thang = thangMoi;
    *nam = namMoi;
    printf("Da xoa toan bo ho so va bat dau ky cham cong %02d/%d.\n",
           *thang, *nam);
    return 1;
}

// note
static int docMaTheoMenu(const char *prompt, char maNV[MA_NV_SIZE]) {
    char maNhap[MA_NV_SIZE + 1];
    int ketQua;

    printf("%s", prompt);
    ketQua = scanf("%20s", maNhap);
    xoa_Xuongdong();

    if (ketQua != 1) {
        printf("Khong doc duoc ma nhan vien.\n");
        return 0;
    }
    if (strlen(maNhap) >= MA_NV_SIZE) {
        printf("Ma nhan vien toi da %d ky tu.\n", MA_NV_SIZE - 1);
        return 0;
    }

    strcpy(maNV, maNhap);
    return 1;
}

static void hienThiMenu(void) {
    printf("\n===== QUAN LY NHAN VIEN =====\n");
    printf("1. Them nhan vien\n");
    printf("2. Sua nhan vien\n");
    printf("3. Xoa nhan vien\n");
    printf("4. Hien thi danh sach\n");
    printf("5. Tim nhan vien theo ma\n");
    printf("6. Cap nhat cham cong\n");
    printf("7. Hien thi bang luong\n");
    printf("8. Sap xep luong giam dan\n");
    printf("9. Tim nhan vien luong cao nhat\n");
    printf("10. Luu du lieu\n");
    printf("11. Luu du lieu va thoat\n");
    printf("12. Chuyen thang va reset cham cong\n");
    printf("13. Xoa toan bo du lieu va bat dau ky moi\n");
}

int main(void) {
    struct NhanVien ds[MAX_NV] = {0};
    int soLuong = 0;
    int thang = 1;
    int nam = 2026;
    int ketQuaDoc = docDuLieu(FILE_NAME, ds, &soLuong, &thang, &nam);
    int luaChon;

    if (ketQuaDoc == 1) {
        printf("Da doc %d nhan vien tu %s. Ky cham cong: %02d/%d.\n",
               soLuong, FILE_NAME, thang, nam);
    } else if (ketQuaDoc == 0) {
        printf("Chua co file du lieu. Bat dau danh sach moi.\n");
        printf("Vui long nhap thang, nam bat dau quan ly.\n");
        if (!nhapThangNam(&thang, &nam)) {
            return 1;
        }
    } else {
        return 1;
    }

    for (;;) {
        hienThiMenu();
        printf("Lua chon: ");

        if (scanf("%d", &luaChon) != 1) {
            if (feof(stdin)) {
                printf("\nKet thuc dau vao; chuong trinh thoat ma khong tu dong luu.\n");
                return 1;
            }
            xoa_Xuongdong();
            printf("Lua chon phai la so nguyen tu tu 1 den 13.\n");
            continue;
        }
        xoa_Xuongdong();

        switch (luaChon) {
            case 1:
                themNhanvien(ds, &soLuong);
                break;
            case 2: {
                char maNV[MA_NV_SIZE];
                if (docMaTheoMenu("Nhap ma nhan vien can sua: ", maNV)) {
                    suaNhanVien(ds, soLuong, maNV);
                }
                break;
            }
            case 3:
                xoaNhanVien(ds, &soLuong);
                break;
            case 4:
                xuatDanhsach(ds, soLuong);
                break;
            case 5: {
                char maNV[MA_NV_SIZE];
                if (docMaTheoMenu("Nhap ma nhan vien can tim: ", maNV)) {
                    timNhanVien(ds, soLuong, maNV);
                }
                break;
            }
            case 6: {
                char maNV[MA_NV_SIZE];
                if (docMaTheoMenu("Nhap ma nhan vien can cap nhat cham cong: ", maNV)) {
                    capNhatChamCong(ds, soLuong, maNV);
                }
                break;
            }
            case 7:
                printf("BANG LUONG THANG %02d/%d\n", thang, nam);
                inBangLuong(ds, soLuong, NGAY_CONG_CHUAN);
                break;
            case 8:
                sapXepLuongGiamDan(ds, soLuong);
                printf("Da sap xep danh sach theo luong giam dan.\n");
                break;
            case 9:
                luong_Best(ds, soLuong);
                break;
            case 10:
                luuDuLieu(FILE_NAME, ds, soLuong, thang, nam);
                break;
            case 11:
                if (luuDuLieu(FILE_NAME, ds, soLuong, thang, nam)) {
                    printf("Da luu du lieu. Tam biet!\n");
                    return 0;
                }
                printf("Luu that bai; chuong trinh van tiep tuc de tranh mat du lieu.\n");
                break;
            case 12:
                if (doiThangChamCong(ds, soLuong, &thang, &nam)) {
                    luuDuLieu(FILE_NAME, ds, soLuong, thang, nam);
                }
                break;
            case 13:
                resetToanBoDuLieu(ds, &soLuong, &thang, &nam);
                break;
            default:
                printf("Vui long chon tu 1 den 13.\n");
                break;
        }
    }
}
