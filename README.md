##sourcode

import javax.swing.*;//JFrame, JPanel, JButton
import java.awt.*;//(layout),(color), font
import java.awt.event.ActionEvent;//digunakan metode actionPerformed
import java.awt.event.ActionListener;//such button
import java.text.SimpleDateFormat;//Memformat objek Date
import java.util.*;//seperti Date, Calendar, List
import java.util.List;//untuk membuat daftar yang dinamis

public class TiketTravelApp {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> { // untuk menghindari masalah threading
            FormPemesanan formPemesanan = new FormPemesanan(); // jendela utama aplikasi
            formPemesanan.setVisible(true); // menampilkan form pemesanan kpd pengguna
        });
    }
}

// menerima input pengguna, menampilkan informasi, dan mengelola logika
class FormPemesanan extends JFrame { // subkelas dari JFrame untuk membuat jendela utama
    private JTextField namaField; // deklarasi komponen (jadi masuk"in aja)
    private JTextField teleponField;
    private JComboBox<String> tujuanComboBox;
    private JComboBox<String> kelasComboBox;
    private JSpinner tanggalSpinner;
    private JSpinner jumlahSpinner;
    private JTextField voucherField;
    private JButton tombolPesan;
    private JButton tombolBayar;
    private JTextArea areaTiket;
    private JLabel totalHargaLabel;
    private int totalHarga = 0; //untuk menyimpan total harga

    private static final String VOUCHER_CODE = "Idha123456";
    private static final double DISCOUNT_RATE = 0.15;
    private static final Map<String, Integer> HARGA_TIKET;

    static {
        HARGA_TIKET = new HashMap<>();
        HARGA_TIKET.put("Jakarta - Bandung", 150000);
        HARGA_TIKET.put("Jakarta - Tasikmalaya", 200000);
        HARGA_TIKET.put("Jakarta - Surabaya", 500000);
        HARGA_TIKET.put("Jakarta - Yogyakarta", 350000);
        HARGA_TIKET.put("Jakarta - Semarang", 300000);
        HARGA_TIKET.put("Bandung - Yogyakarta", 300000);
        HARGA_TIKET.put("Surabaya - Solo", 150000);
        HARGA_TIKET.put("Jakarta - Bali", 600000);
    }

    public FormPemesanan() {
        setTitle("Pemesanan Tiket Travel"); // judul
        setSize(600, 700); // ukuran
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // saat keluar apl
        setLocationRelativeTo(null); // tampilan di tengah

        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(10, 2)); // tata letak grid 10,2

        // Label dan bidang
        JLabel namaLabel = new JLabel("Nama:");
        namaField = new JTextField();
        JLabel teleponLabel = new JLabel("No. Telepon:");
        teleponField = new JTextField();
        JLabel tujuanLabel = new JLabel("Tujuan:");
        tujuanComboBox = new JComboBox<>(HARGA_TIKET.keySet().toArray(new String[0]));
        JLabel kelasLabel = new JLabel("Kelas:");
        String[] kelas = { "Ekonomi", "Bisnis", "VIP" };
        kelasComboBox = new JComboBox<>(kelas);

        JLabel tanggalLabel = new JLabel("Tanggal Keberangkatan:");
        SpinnerDateModel dateModel = new SpinnerDateModel(new Date(), null, null, Calendar.DAY_OF_MONTH);
        tanggalSpinner = new JSpinner(dateModel);
        JSpinner.DateEditor dateEditor = new JSpinner.DateEditor(tanggalSpinner, "yyyy-MM-dd");
        tanggalSpinner.setEditor(dateEditor);

        JLabel jumlahLabel = new JLabel("Jumlah Tiket:");
        jumlahSpinner = new JSpinner(new SpinnerNumberModel(1, 1, 10, 1));

        JLabel voucherLabel = new JLabel("Kode Voucher:");
        voucherField = new JTextField();

        tombolPesan = new JButton("Pesan Tiket");
        tombolBayar = new JButton("Bayar");
        areaTiket = new JTextArea(); // untuk menampilkan informasi tiket yang dipesan
        areaTiket.setEditable(false);
        totalHargaLabel = new JLabel("Total Harga: Rp 0"); // mulai dari 0

        // pengelompokan agar lebih mudah teratur
        panel.add(namaLabel);
        panel.add(namaField);
        panel.add(teleponLabel);
        panel.add(teleponField);
        panel.add(tujuanLabel);
        panel.add(tujuanComboBox);
        panel.add(kelasLabel);
        panel.add(kelasComboBox);
        panel.add(tanggalLabel);
        panel.add(tanggalSpinner);
        panel.add(jumlahLabel);
        panel.add(jumlahSpinner);
        panel.add(voucherLabel);
        panel.add(voucherField);
        panel.add(new JLabel());
        panel.add(tombolPesan);
        panel.add(new JLabel());
        panel.add(totalHargaLabel);
        panel.add(new JLabel());
        panel.add(tombolBayar);

        add(panel, BorderLayout.CENTER);
        add(new JScrollPane(areaTiket), BorderLayout.SOUTH);

        // membantu membuat aplikasi lebih interaktif dan responsif terhadap input
        // pengguna.
        tombolPesan.addActionListener(new TombolPesanListener());
        tombolBayar.addActionListener(new TombolBayarListener());
    }

    // mengambil data dari form, memvalidasi input, menghitung harga tiket,
    // menyimpan tiket ke database, dan menampilkan informasi tiket di area teks.
    private class TombolPesanListener implements ActionListener {
        @Override
        // data di ambil dari form input di bawah
        public void actionPerformed(ActionEvent e) {
            String nama = namaField.getText();
            String telepon = teleponField.getText();
            String tujuan = (String) tujuanComboBox.getSelectedItem();
            String kelas = (String) kelasComboBox.getSelectedItem();
            Date tanggal = (Date) tanggalSpinner.getValue();
            int jumlahTiket = (int) jumlahSpinner.getValue();
            String voucher = voucherField.getText();

            // memastikan agar semua file diisi, kalo belum gabisa lanjut
            if (nama.isEmpty() || telepon.isEmpty() || tujuan == null || kelas == null || tanggal == null) {
                JOptionPane.showMessageDialog(FormPemesanan.this, "Silakan masukkan semua informasi.", "Error",
                        JOptionPane.ERROR_MESSAGE);
                return;
            }

            // harga dasarnya diambil dari harga awal, harga akhirnya diambil setelah beres
            // semua diitung dan jika ada voucher yang valid.
            int hargaDasar = HARGA_TIKET.get(tujuan);
            int hargaPerTiket = hitungHargaBerdasarkanKelas(hargaDasar, kelas);

            boolean isVoucherValid = VOUCHER_CODE.equals(voucher);
            if (isVoucherValid) {
                hargaPerTiket -= hargaPerTiket * DISCOUNT_RATE;
            }

            int totalHargaPesanan = hargaPerTiket * jumlahTiket;
            totalHarga += totalHargaPesanan;

            // semua tiket disimpan di database, lalu lalu ditampilkan di area teks
            // jika ada voucher valid, maka ditampilkan di area teks juga
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
            String tanggalStr = dateFormat.format(tanggal);

            for (int i = 0; i < jumlahTiket; i++) {
                Tiket tiket = new Tiket(nama, telepon, tujuan, kelas, tanggal, hargaPerTiket);
                DatabaseTiket.simpanTiket(tiket);
                areaTiket.append("Tiket dipesan untuk " + nama + " ke " + tujuan + " (Kelas: " + kelas + ", Tanggal: "
                        + tanggalStr + ", No. Telepon: " + telepon + ") dengan harga Rp" + hargaPerTiket);
                if (isVoucherValid) {
                    areaTiket.append(" (diskon 15%)");
                }
                areaTiket.append(".\n");
            }
            totalHargaLabel.setText("Total Harga: Rp " + totalHarga);

            // Setelah pemesanan berhasil, form direset ke keadaan awal untuk pemesanan
            // baru.
            namaField.setText("");
            teleponField.setText("");
            tujuanComboBox.setSelectedIndex(0);
            kelasComboBox.setSelectedIndex(0);
            tanggalSpinner.setValue(new Date());
            jumlahSpinner.setValue(1);
            voucherField.setText("");
        }

        // menghitung harga tiket berdasarkan kelas yang dipilih.
        // parameter -harga dasar -kelas (bisnis, vip)
        private int hitungHargaBerdasarkanKelas(int hargaDasar, String kelas) {
            switch (kelas) {
                case "Bisnis":
                    return hargaDasar + 80000;
                case "VIP":
                    return hargaDasar + 160000;
                default:
                    return hargaDasar;
            }
        }
    }

    // menangani aksi saat tombol "Bayar" ditekan
    // jika 0 berarti salah
    // jika berhasil maka ada tulisan berhasil pembayaran, lalu total harga dll nya
    // direset dan dikosongkan seperti semula
    private class TombolBayarListener implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            if (totalHarga == 0) {
                JOptionPane.showMessageDialog(FormPemesanan.this, "Tidak ada tiket yang dipesan.", "Error",
                        JOptionPane.ERROR_MESSAGE);
                return;
            }

            JOptionPane.showMessageDialog(FormPemesanan.this, "Pembayaran sebesar Rp " + totalHarga + " berhasil.",
                    "Pembayaran Berhasil", JOptionPane.INFORMATION_MESSAGE);
            totalHarga = 0;
            totalHargaLabel.setText("Total Harga: Rp 0");
            areaTiket.setText("");
            DatabaseTiket.kosongkanTiket();
        }
    }
}

// merepresentasikan tiket perjalanan dengan beberapa atribut di bawah
class Tiket {
    private String nama;
    private String telepon;
    private String tujuan;
    private String kelas;
    private Date tanggal;
    private int harga;

    public Tiket(String nama, String telepon, String tujuan, String kelas, Date tanggal, int harga) {
        this.nama = nama;
        this.telepon = telepon;
        this.tujuan = tujuan;
        this.kelas = kelas;
        this.tanggal = tanggal;
        this.harga = harga;
    }

    // METODE GETTER
    // untuk mengembalikan nilai dari atribut-atribut private dalam sebuah kelas.
    public String getNama() {
        return nama;
    }

    public String getTelepon() {
        return telepon;
    }

    public String getTujuan() {
        return tujuan;
    }

    public String getKelas() {
        return kelas;
    }

    public Date getTanggal() {
        return tanggal;
    }

    public int getHarga() {
        return harga;
    }
}

// untuk menyimpan dan mengelola daftar objek Tiket yang telah dibuat
class DatabaseTiket {
    private static List<Tiket> tiketList = new ArrayList<>();

    // untuk menambahkan objek Tiket baru ke dalam tiketList
    public static void simpanTiket(Tiket tiket) {
        tiketList.add(tiket);
    }

    // mengakses semua objek Tiket yang tersimpan tanpa mengubah struktur internal
    // tiketList
    public static List<Tiket> getTiket() {
        return new ArrayList<>(tiketList);
    }

    // untuk menghapus semua objek Tiket yang tersimpan di dalam tiketList
    public static void kosongkanTiket() {
        tiketList.clear();
    }
}

##output

![image](https://github.com/idhahamidaturrosadi19/Pa-Irfan_UAS/assets/144808574/d12c30ba-b24b-4329-99db-8cced974bd72)
![image](https://github.com/idhahamidaturrosadi19/Pa-Irfan_UAS/assets/144808574/0a1a30e4-b4d5-42ec-b05b-5075b4c87f64)
![image](https://github.com/idhahamidaturrosadi19/Pa-Irfan_UAS/assets/144808574/5567fc5d-f737-4380-b4b4-9ab25c866abb)
![image](https://github.com/idhahamidaturrosadi19/Pa-Irfan_UAS/assets/144808574/8d4d04f1-2074-4d37-b1a2-b97317d74eb2)
![image](https://github.com/idhahamidaturrosadi19/Pa-Irfan_UAS/assets/144808574/3549b9d5-16e5-464f-b16f-3136fb0ad0b9)




