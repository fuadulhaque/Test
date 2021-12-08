# Test
Create Repository for test
namespace ERP.Infrastructure.Utils.Repositories
{
    public abstract class AsyncRepository<T> : IAsyncRepository<T> where T : BaseEntity, IAggregateRoot
    {
        protected readonly DbContext _dbContext;

        public IDatabaseManager DatabaseManager { get; }

        protected AsyncRepository(DbContext dbContext)
        {
            _dbContext = dbContext;
            _dbContext.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
            DatabaseManager = new DatabaseManager<DbContext>(dbContext);
        }

        public async Task<T> GetByIdAsync(long id)
        {
            return await _dbContext.Set<T>().FindAsync(id);
        }

        public async Task<IList<T>> ListAsync()
        {
            return await _dbContext.Set<T>().ToListAsync();
        }

        public async Task<IList<T>> ListAsync(Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            return await _dbContext.Set<T>().AsNoTracking().Specify(criteria, navigations).ToListAsync();
        }

        public async Task<T> FirstOrDefaultAsync()
        {
            var result = await _dbContext.Set<T>().AsNoTracking().ToListAsync();
            return result.FirstOrDefault();
        }

        public async Task<T> FirstOrDefaultAsync(Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            var result = await _dbContext.Set<T>().AsNoTracking().Specify(criteria, navigations).ToListAsync();
            return result.FirstOrDefault();
        }

        public async Task<T> LastOrDefaultAsync()
        {
            var result = await _dbContext.Set<T>().AsNoTracking().ToListAsync();
            return result.LastOrDefault();
        }

        public async Task<T> LastOrDefaultAsync(Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            var result = await _dbContext.Set<T>().AsNoTracking().Specify(criteria, navigations).ToListAsync();
            return result.LastOrDefault();
        }

        public async Task<string> MaxAsync(Expression<Func<T, string>> maxBy, Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            return await _dbContext.Set<T>().Specify(criteria, navigations).MaxAsync(maxBy);
        }

        public async Task<string> NextAsync(Expression<Func<T, string>> itemName, int itemLength, Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            var maxValue = await _dbContext.Set<T>().Specify(criteria, navigations).MaxAsync(itemName);
            int nextValue = 0;

            if (!string.IsNullOrWhiteSpace(maxValue))
            {
                nextValue = Convert.ToInt32(maxValue) + 1;
            }

            string nexVal = Convert.ToString(nextValue);
            return nexVal.Length > itemLength ? string.Empty : nexVal.PadLeft(itemLength, '0');
        }

        public async Task<int> CountAsync(Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            return await _dbContext.Set<T>().Specify(criteria, navigations).CountAsync();
        }

        public async Task<bool> AnyAsync(Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            return await _dbContext.Set<T>().Specify(criteria, navigations).AnyAsync();
        }

        public async Task<IList<T>> ListAsync(ISpecification<T> spec)
        {
            return await ApplySpecification(spec).ToListAsync();
        }

        public async Task<IList<T>> ListAsyncSpecify(ISpecification<T> spec)
        {
            return await _dbContext.Set<T>().Specify(spec).ToListAsync();
        }

        public async Task<T> FirstOrDefaultAsync(ISpecification<T> spec)
        {
            return await ApplySpecification(spec).AsNoTracking().FirstOrDefaultAsync();
        }

        //public async Task<IList<T>> ListSpecThenAsync(ISpecification<T> spec, ISpecification<T> specThen)
        //{
        //    return await _dbContext.Set<T>().Specify(spec).Specify(specThen).ToListAsync();
        //}

        public async Task<int> CountAsync(ISpecification<T> spec)
        {
            return await ApplySpecification(spec).CountAsync();
        }

        public async Task<string> MaxAsync(Expression<Func<T, string>> maxBy, ISpecification<T> spec)
        {
            return await ApplySpecification(spec).MaxAsync(maxBy);
        }

        public async Task<bool> AnyAsync(ISpecification<T> spec)
        {
            return await ApplySpecification(spec).AnyAsync();
        }

        public async Task<T> AddAsync(T entity)
        {
            try
            {
                _dbContext.Set<T>().Add(entity);
                await _dbContext.SaveChangesAsync();
                return entity;
            }
            catch (Exception ex)
            {
                throw ex;
            }
        }

        //public async Task AddListAsync(IList<T> entityList)
        //{
        //    foreach (var entity in entityList)
        //    {
        //        _dbContext.Set<T>().Add(entity);
        //    }
        //    await _dbContext.SaveChangesAsync();
        //}    

        public async Task<IList<T>> AddListAsync(IList<T> entityList)
        {
            _dbContext.Set<T>().AddRange(entityList);
            await _dbContext.SaveChangesAsync();
            return entityList;
        }

        public async Task UpdateAsync(T entity)
        {
            try
            {
                _dbContext.Entry(entity).State = EntityState.Modified;
                await _dbContext.SaveChangesAsync();
            }
            catch (Exception exception)
            {
                throw exception;
            }
        }

        public async Task UpdateAsync(T entity, Expression<Func<T, object>>[] properties)
        {
            _dbContext.Set<T>().Attach(entity);

            foreach (var property in properties)
            {
                _dbContext.Entry(entity).Property(property).IsModified = true;
            }
            await _dbContext.SaveChangesAsync();
        }

        public async Task UpdateExceptAsync(T entity, Expression<Func<T, object>>[] properties)
        {
            _dbContext.Set<T>().Attach(entity);
            
            foreach (var property in properties)
            {
                _dbContext.Entry(entity).Property(property).IsModified = false;
            }
            await _dbContext.SaveChangesAsync();
        }
        public async Task UpdateExceptOneAsync(T entity, Expression<Func<T, object>>[] properties)
        {
            _dbContext.Set<T>().Attach(entity);
            
            foreach (var property in properties)
            {
                _dbContext.Entry(entity).State = EntityState.Modified;
                _dbContext.Entry(entity).Property(property).IsModified = false;
            }
            await _dbContext.SaveChangesAsync();
        }

        public async Task UpdateListAsync(IList<T> entityList)
        {
            foreach (var entity in entityList)
            {
                _dbContext.Entry(entity).State = EntityState.Modified;
            }
            await _dbContext.SaveChangesAsync();
        }

        public async Task DeleteAsync(T entity)
        {
            var dbEntity = await GetByIdAsync((long)entity.Id);
            if (dbEntity != null)
            {
                dbEntity.IsDeleted = true;
                dbEntity.IsActive = false;
                await UpdateAsync(dbEntity);
            }
        }

        public async Task PermanentDeleteAsync(T entity)
        {
            _dbContext.Set<T>().Remove(entity);
            await _dbContext.SaveChangesAsync();
        }

        public async Task PermanentDeleteRangeAsync(IList<T> entity)
        {
            _dbContext.Set<T>().RemoveRange(entity);
            await _dbContext.SaveChangesAsync();
        }

        private IQueryable<T> ApplySpecification(ISpecification<T> spec)
        {
            return SpecificationEvaluator<T>.GetQuery(_dbContext.Set<T>().AsNoTracking().AsQueryable(), spec);
        }

        //public async Task<IList<T>> GetListFromStoredProcedure(string procedureName, bool isActive)
        //{
        //    var result = await _dbContext.Set<T>().FromSqlRaw($"EXEC {procedureName} {isActive}").ToListAsync();
        //    return result;
        //}

        public async Task<IList<T>> GetListFromStoredProcedure(string procedureName, params SqlParameter[] listParameters)
        {
            try
            {
                var query = $"EXEC {procedureName} ";
                query = listParameters.Aggregate(query, (current, parameter) => parameter.DbType == DbType.String
                    ? $"{current}'{parameter.Value}',"
                    : $"{current}{parameter.Value},");

                query = query.Substring(0, query.Length - 1);

                string finalSql = query.Contains("''")
                    ? query.Replace("''", "NULL")
                    : query;

                var result = await _dbContext.Set<T>().FromSqlRaw(finalSql).ToListAsync();
                return result;
            }
            catch (Exception ex)
            {
                throw ex;
            }
        }

        public async Task<IList<T>> GetListFromStoredProcedure(string procedureName, object viewModel)
        {
            try
            {
                var listParameters = Utility.GetSqlParameterList(viewModel);

                var query = $"SET ARITHABORT ON \n EXEC {procedureName} ";
                foreach (var parameter in listParameters)
                {
                    var parameterValue = parameter.DbType == DbType.String
                        ? $"'{parameter.Value}'"
                        : $"{parameter.Value}";
                    query += $"{parameter.ParameterName} = {parameterValue},";
                }

                query = query.Substring(0, query.Length - 1);

                string finalSql = query.Contains("''")
                    ? query.Replace("''", "NULL")
                    : query;

                var result = await _dbContext.Set<T>().FromSqlRaw(finalSql).ToListAsync();
                return result;
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
                throw;
            }
        }

        public async Task<bool> IsDuplicateAsync(long? id, Expression<Func<T, bool>> criteria, params string[] navigations)
        {
            if (criteria != null & id > 0)
            {
                return await _dbContext.Set<T>().Specify(criteria, navigations).AsNoTracking().Where(s => !s.IsDeleted && s.IsActive && s.Id != id).AnyAsync();
            }

            return await _dbContext.Set<T>().Specify(criteria, navigations).AsNoTracking().AnyAsync();
        }
    }
}
  namespace ERP.Infrastructure.Utils.Repositories
{
    public class DatabaseManager<T> : IDatabaseManager where T : DbContext
    {
        public SqlConnection _dbConnection;
        public T _dbContext;
        int timeOut;

        public DatabaseManager(T dbContext)
        {
            _dbContext = dbContext;
            _dbConnection = new SqlConnection(_dbContext.Database.GetDbConnection().ConnectionString);
            timeOut = 360000;
        }

        public async Task<SqlConnection> OpenConnectionAsync()
        {
            if (_dbConnection.State == ConnectionState.Closed)
            {
                await _dbConnection.OpenAsync();
            }
            return _dbConnection;
        }

        public async Task CloseConnectionAsync()
        {
            if (_dbConnection.State == ConnectionState.Open)
            {
                await _dbConnection.CloseAsync();
            }
        }


        public async Task<string> ExecuteScalarAsync(string query)
        {
            try
            {
                //LogHelper.WriteDbSelectInfo(query);

                _dbConnection = await OpenConnectionAsync();
                var command = new SqlCommand()
                {
                    Connection = _dbConnection,
                    CommandTimeout = timeOut,
                    CommandType = CommandType.Text,
                    CommandText = query
                };

                var result = await command.ExecuteScalarAsync();
                if (result != null) { 
                return result.ToString();
                }
                else
                {
                    return null;
                }
            }
            catch (Exception exception)
            {
                //LogHelper.WriteDbException(exception);
                return "";
            }
            finally
            {
                await CloseConnectionAsync();
            }
        }

        public async Task<string> ExecuteScalarAsync(string tableName, string columnName, string clauseName, string clauseValue)
        {
            var query = $"SELECT {columnName} FROM {tableName} WHERE {clauseName} = '{clauseValue}'";
            var result = await ExecuteScalarAsync(query);
            return result;
        }

        public async Task<string> ExecuteScalarAsyncForInClause(string tableName, string columnName, string clauseName, string clauseValue)
        {
            string clauseString = "";

            var stringList = clauseValue.Split(",");
            foreach (var item in stringList)
            {
                clauseString =$"'{item}',";
            }
            clauseString = clauseString.Substring(0, clauseString.Length - 1);

            var query = $"SELECT {columnName} FROM {tableName} WHERE {clauseName} IN ({clauseString})";
            var result = await ExecuteScalarAsync(query);
            return result;
        }

        public async Task<DataTable> ExecuteQueryForDataTableAsync(string query)
        {
            var dataTable = new DataTable();
            try
            {
                //LogHelper.WriteDbSelectInfo(query);

                _dbConnection = await OpenConnectionAsync();
                var command = new SqlCommand()
                {
                    Connection = _dbConnection,
                    CommandTimeout = timeOut,
                    CommandType = CommandType.Text,
                    CommandText = query
                };

                var dataAdapter = new SqlDataAdapter(command);
                dataAdapter.Fill(dataTable);
                return dataTable;
            }
            catch (Exception exception)
            {
                throw exception;
                //LogHelper.WriteDbException(exception);
                //return null;
            }
            finally
            {
                await CloseConnectionAsync();
            }
        }

        public async Task<DataSet> ExecuteQueryForDataSetAsync(string query)
        {
            try
            {
                //LogHelper.WriteDbSelectInfo(query);

                var dataSet = new DataSet();
                _dbConnection = await OpenConnectionAsync();
                var command = new SqlCommand()
                {
                    Connection = _dbConnection,
                    CommandTimeout = timeOut,
                    CommandType = CommandType.Text,
                    CommandText = query
                };

                var adapter = new SqlDataAdapter(command);
                adapter.Fill(dataSet);
                return dataSet;
            }
            catch (Exception exception)
            {
                //LogHelper.WriteDbException(exception);
                return null;
            }
            finally
            {
                await CloseConnectionAsync();
            }
        }

        public async Task<int> ExecuteNonQueryAsync(string tableName, string columnName, object columnValue, string clauseName, string clauseValue)
        {
            //string clauseString = "";

            //var stringList = clauseValue.Split(",");
            //foreach (var item in stringList)
            //{
            //    clauseString = $"'{item}',";
            //}
            //clauseString = clauseString.Substring(0, clauseString.Length - 1);

            var query = $"UPDATE {tableName} SET  {columnName} = {columnValue} WHERE {clauseName} IN ({clauseValue})";
            return await ExecuteNonQueryAsync(query);
        }

        public async Task<int> ExecuteNonQueryAsync(string query)
        {
            try
            {
                //LogHelper.WriteDbInsertInfo(query);

                _dbConnection = await OpenConnectionAsync();
                var command = new SqlCommand()
                {
                    Connection = _dbConnection,
                    CommandTimeout = timeOut,
                    CommandType = CommandType.Text,
                    CommandText = query
                };

                return await command.ExecuteNonQueryAsync();
            }
            catch (Exception exception)
            {
                //LogHelper.WriteDbException(exception);
                return 0;
            }
            finally
            {
                await CloseConnectionAsync();
            }
        }
    }
}
