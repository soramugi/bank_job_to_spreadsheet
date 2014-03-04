require 'bundler'
require 'yaml'
Bundler.require

namespace :bank do
  desc '一覧'
  task :list do
    p bank_list
  end

  desc 'Google スプレッドシートに入力'
  task :spredseet do
    google_drive_conf = config['google_drive']
    session = GoogleDrive.login(google_drive_conf['mail'], google_drive_conf['password'])
    # First worksheet of
    ws = session.spreadsheet_by_key(google_drive_conf['spreadsheet_by_key']).worksheets[0]

    total = 0
    count = 2
    ws[ws.num_cols + 1, 1] = Time.now.strftime('%Y/%m/%d')
    bank_list.each do |list|
      deposits = list[:deposits].gsub(%r{円|,},"").to_i
      ws[ws.num_cols + 1, count] = deposits
      count += 1
      total += deposits
    end
    ws[ws.num_cols + 1, count] = total
    ws.save
    ws.reload
  end
end

def config
  @config ||= YAML.load_file(File.expand_path('~/.bank_job.yml'))
end

def bank_list
  config['registers'].map do |register|
    strategy = register['strategy']
    register.delete('strategy')
    bj = BankJob.new
    bj.register do |bank|
      eval("require 'bank_job_#{strategy.downcase}'")
      bank.strategy = eval("BankJob::Strategy::#{strategy}.new")
      register.each do |k,v|
        bank.send("#{k}=", v)
      end
    end
    {
      strategy: strategy,
      deposits: bj.agents.first.deposits,
    }
  end
end
